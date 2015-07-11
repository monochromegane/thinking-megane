---
title: "Android Studioを使うときに調べたこと、いくつか"
date: 2013-07-08
comments: true
tags: [ "android", "studio", "java" ]
---
ひさびさにAndroidアプリをつくるのに、せっかくなので公開された[Android Studio](http://developer.android.com/sdk/installing/studio.html)を使ってみました。

Eclipseでの開発環境と違いなど、調べたことがいくつかあったのでまとめておきます。

![Android Studio](/images/2013/07/android_studio.png) 

今回調べたのは以下の4つです。

1. assetsディレクトリが作成されない
2. エミュレータでプロキシを使いたい
3. 日本語コメントを入れるとコンパイルできない
4. ライブラリ(jar)追加でコンパイルできない

なお、環境は2013/07/08時点での最新版、`Android Studio (I/O Preview) AI-130.729444`です。

<hr />

# 1. assetsディレクトリが作成されない

Android Studioではプロジェクト作成時にassetsディレクトリが作成されませんでした。

また、assetsディレクトリを作成する場合、`src/main`の下に作成しないといけませんでした。

### どこで定義されている？

Eclipseでの開発時はassetsディレクトリはプロジェクト直下にありますが、Android Studioではそこにディレクトリを作成してもアプリから認識されません。

これは、モジュール定義ファイル(.iml)に以下のように定義されているためです。

```xml
<sourceFolder url="file://$MODULE_DIR$/src/main/assets" isTestSource="false" />
```

<hr />
# 2. エミュレータでプロキシを使いたい

[Run]メニュー > [Edit Configurations...]を選択し、開いたウィンドウから[Emulator]タブを選択します。

[Additional command line options:]にチェックを入れ、以下のコマンドラインオプションを追加します。

```
-http-proxy http://hostname:port
```

<hr />
# 3. 日本語コメントを入れるとコンパイルできない

Android Studioで、Javaソース内に日本語でコメントを入れると以下のエラーが出てコンパイルに失敗します。

```
この文字は、エンコーディングsjisにマップできません。
```

VMのオプションを設定ファイルに追記する必要があるようです。


`/Applications/Android Studio.app/Contents/Info.plist`
```
<key>VMOptions</key>
<string>-ea -Dsun.io.useCanonCaches=false -XX:+HeapDumpOnOutOfMemoryError -Xverify:none -Xbootclasspath/a:../lib/boot.jar -Dgroovy.source.encoding=UTF-8 -Dfile.encoding=UTF-8</string>
```
`-Dgroovy.source.encoding=UTF-8 -Dfile.encoding=UTF-8`を追加しています。
追加後にAndroid Studioを再起動すればOKです。


*こちらを参考にさせていただきました。

[takatoshi blog / [AndroidStudio]この文字は、エンコーディングsjisにマップできません。コンパイルエラーへの対処](http://takatoshimaeda.github.io/blog/2013/05/20/android-studio-character-code-build-error/)


<hr />
# 4. ライブラリ(jar)追加でコンパイルできない

ライブラリを追加した場合、以下の2つの設定を行なう必要がありました。

特に`実行時の設定`がない場合、実行時に以下のエラーが発生します。
```
パッケージ xxx は存在しません
```

### コンパイルのための設定

1. プロジェクト直下のlibsディレクトリにjarファイルを配置する。
2. Android Studioで該当のjarファイルを右クリック > Add as Library... を行う

### 実行時の設定

`build.gradle`
```
dependencies {
    compile files('libs/android-support-v4.jar')
    compile files('libs/your-library.jar')
}
```
`dependencies`内にjarファイル名を追加します。

<hr />

以上、いくつか調べることはありましたがAndroid Studio、なかなか快適です。

Eclipseで開発している方であれば、比較的すんなり移行できると思います。

設定ファイルまわりの編集作業はAndroid Studioのバージョンが上がれば不要になってくる作業と思いますが、現時点ではこういう設定もあるということを覚えておいてもいいかもしれません。

以上です。
