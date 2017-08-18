+++
date = "2017-08-18T18:13:52+09:00"
title = "SSD: Single Shot MultiBox DetectorをGoogle Cloud ML Engine上で物体検出APIとして利用する"
tags = [ "機械学習" ]
+++

画像内の物体検出と識別を行う予測APIが必要になったので、物体検出ニューラルネットワークである[SSD: Single Shot MultiBox Detector](http://arxiv.org/abs/1512.02325)を[Google Cloud ML Engine](https://cloud.google.com/products/machine-learning/)上で訓練して予測APIとして使えるようにしました。

<iframe src="//hatenablog-parts.com/embed?url=https%3A%2F%2Fgithub.com%2Fmonochromegane%2Fssd_mlengine" title="monochromegane/ssd_mlengine" class="embed-card embed-webcard" scrolling="no" frame border="0" style="width: 100%; height: 155px; max-width: 500px; margin: 10px 0px;">&lt;a href="https://github.com/monochromegane/ssd_mlengine"&gt;monochromegane/ssd_mlengine&lt;/a&gt;</iframe>

## 使い方

Google Cloud ML Engineでモデル、学習時のパラメタ、予測APIのバージョンなどをコード管理するため、[StarChart](https://github.com/monochromegane/starchart)を使います。
コード管理が不要であれば、リポジトリのコードをCloud Storageにアップロードして`gcloud`コマンドなどで訓練を行ってください。
訓練データセットなどをCloud Storageに配置した上で、以下のようにして訓練用のジョブを投入します。

```sh
$ pip install git+https://github.com/monochromegane/starchart.git
$ git clone https://github.com/monochromegane/ssd_mlengine.git
$ starchart train -m ssd_mlengine \
                  -M trainer.task \
                  -s BASIC_GPU \
                  -- \
                  --annotation_path=gs://PATH_TO_ANNOTATION_FILE \
                  --prior_path=gs://PATH_TO_PRIOR_FILE \
                  --weight_path=gs://PATH_TO_WEIGHT_FILE \
                  --images_path=gs://PATH_TO_IMAGES_FILE \
                  --model_dir=TRAIN_PATH/model
```

訓練が終わったら、以下のようにしてモデルを予測APIとして公開します。

```
$ starchart expose -m ssd_mlengine
```

公開された予測APIを叩くと結果が返ります。

```
$ python predict.py -k 1 -c 0.45 -i image.jpg
# {'predictions': [{'key': '1',
#    'objects': [[8.0,        # 検出した物体の分類区分
#      0.45196059346199036,   # 確率
#      104.90727233886719,    # 検出した位置(xmin)
#      97.99836730957031,     # 検出した位置(ymin)
#      212.12222290039062,    # 検出した位置(xmax)
#      315.3045349121094]]}]} # 検出した位置(ymax)
```

予測APIを叩くコードはREADMEを参考にしてください。パラメタは以下の通りです。

- keep\_top\_k(-k): 確率の降順でソートされたうち、上位何件を取得するか
- confidence\_threshold(-c): 確率の閾値


## 実装

SSDのKeras実装である[rykov8/ssd_keras](https://github.com/rykov8/ssd_keras)を参考にGoogle Cloud ML Engineで訓練、予測APIとして利用できるようにしています。

SSDはモデルの出力をそのまま画像内の検出した位置として利用できないため、元画像で利用できるよう結果を変換する必要がありますが、ちと面倒なのでこの部分もAPI側で行うようにします。
このような予測APIだけ必要となる変換層はKerasとML Engineを利用する場合、以下のようにして追加できます。

```py
# keras.layers.Lambdaで学習モデルの出力を入力として変換する層を定義
detection_out = Lambda(bbox_util.detection_out, arguments={
    'keep_top_k': keep_top_k_placeholder,
    'confidence_threshold': confidence_threshold_placeholder,
    'original_size': original_size_placeholder
    })(net['predictions'])

# 上で定義したLambdaを予測APIの出力として定義
inputs  = {'key': keys_placeholder,
           'data': model.input,
           'keep_top_k': keep_top_k_placeholder,
           'confidence_threshold': confidence_threshold_placeholder,
           'original_size': original_size_placeholder}
outputs = {'key': tf.identity(keys_placeholder), 'objects': detection_out}
```

これらの定義をSavedModel形式で保存することでML Engineが予測APIとして扱えるようにします

```py
def build_signature(inputs, outputs):
    signature_inputs = {key: saved_model_utils.build_tensor_info(tensor)
                        for key, tensor in inputs.items()}
    signature_outputs = {key: saved_model_utils.build_tensor_info(tensor)
                         for key, tensor in outputs.items()}

    signature_def = signature_def_utils.build_signature_def(
        signature_inputs, signature_outputs,
        signature_constants.PREDICT_METHOD_NAME)

    return signature_def

def export(sess, inputs, outputs, output_dir):
    if file_io.file_exists(output_dir):
        file_io.delete_recursively(output_dir)

    signature_def = build_signature(inputs, outputs)

    signature_def_map = {
        signature_constants.DEFAULT_SERVING_SIGNATURE_DEF_KEY: signature_def
    }
    builder = saved_model_builder.SavedModelBuilder(output_dir)
    builder.add_meta_graph_and_variables(
        sess,
        tags=[tag_constants.SERVING],
        signature_def_map=signature_def_map)

    builder.save()

# SavedModel形式で保存する
export(get_session(), inputs, outputs, FLAGS.model_dir)
```

なお、今回は、ML Engineの保存ファイルが256MB以内とする制限を回避するため、ベースネットワークのconv4~pool4層も固定し、その出力を元に38x38分割したサイズの物体を検出するレイヤも除去することで学習パラメタを削減しています。論文では出力レイヤを減らすと精度が低下するとあるので、このあたりは対象のドメインや要求する精度によって調整が必要かもしれません。


## 学習に必要となるデータなど

学習済みの重みやバイアスを保存した `weights_SSD300.hdf5` と、計算済みのデフォルトボックスの位置を保存した `prior_boxes_ssd300.pkl` を [rykov8/ssd_keras](https://github.com/rykov8/ssd_keras)から入手しておきます。

前述の通り、デフォルトボックスを除去しているので、

```py
import pickle
pickle.load(open('prior_boxes_ssd300.pkl', 'rb'))[4332:].dump('prior_boxes_ssd300_min.pkl')
```

のようにして、必要なものだけを取り出しておく必要があります。


また、教師データとして、画像ならびにその画像に対する物体の位置と分類クラスを定義したデータセットが必要です。今回はrykov8/ssd_kerasと同じくVOCのデータセット形式を採用しています。
独自にデータセットを準備できたら、[rykov8/ssd_kerasの提供しているツール](https://github.com/rykov8/ssd_keras/blob/master/PASCAL_VOC/get_data_from_XML.py)を使ってpickle形式で保存したものをannotation_pathとして、もととなった画像データをtar.gzで固めたものをimages_pathとして指定します。

## まとめ

物体検出ニューラルネットワークであるSSDのKeras実装をGoogle Cloud ML Engine上で訓練し、予測APIとして提供できるようにしました。実際にAPIとして運用するにあたってはモデル、学習時のパラメタ、予測APIのバージョンなどをコード管理が求められることになるため、[StarChart](https://github.com/monochromegane/starchart)のようなツールを検討も大切だと思います。StarChartについては[こちら](https://rand.pepabo.com/article/2017/01/18/pepabo-ml-platform-and-workflow/)や[こちら](https://speakerdeck.com/monochromegane/pepabo-ml-infrastructure-starchart)でも紹介しています。よければ参考にしてください。

