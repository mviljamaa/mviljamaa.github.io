---
layout: page
status: publish
published: true
title: 'チュートリアル10:透明'
date: '2011-05-13 23:00:42 +0200'
date_gmt: '2011-05-13 23:00:42 +0200'
categories: [tuto]
order: 20
tags: []
language: jp
---

#アルファチャネル

アルファチャネルの概念はとても単純です。RGBを書くの代わりに、RGBAを書きます：
{% highlight glsl linenos cssclass=highlightglslfs %}
// 出力データ：今回はvec4です。
out vec4 color;
{% endhighlight %}
最初の3要素は.xyzでアクセスでき、最後の要素は.aでアクセスします：
{% highlight glsl linenos cssclass=highlightglslfs %}
color.a = 0.3;
{% endhighlight %}
直感的ではありませんが、alpha=不透明度、そのためalpha=1は完全な不透明を、一方でalpha=0は完全な透明を表します。

ここで、単純にアルファチャネルを0.3と直書きしていますが、きっとRGBAテクスチャから読み込んだ値を使いたいでしょう。（TGAはアルファチャネルをサポートしており、GLFWはTGAをサポートしています。）

メッシュの中を見るときに、"背面"がないものとしたいので、バックフェースカリングをオフにしました。 (glDisable(GL_CULL_FACE) )

![]({{site.baseurl}}/assets/images/tuto-10-transparency/transparencyok.png)


#順序について！

上で示した結果はちゃんと表示されてますが、それは単に運が良かっただけです。

##問題

ここで、50％アルファの緑と赤の二つの四角形を描画するとします。 下の図を見れば順序が重要であることが見てわかるでしょう。つまり、最終的な色はどちらが前面にあるかを判断する重要なカギとなります。

![]({{site.baseurl}}/assets/images/tuto-10-transparency/transparencyorder.png)


同様のことが視点を少し変えた下のシーンでも起こっています：

![]({{site.baseurl}}/assets/images/tuto-10-transparency/transparencybad.png)


これはとても面倒な問題だとわかったでしょう。実際ゲームではあまり透明なものは見ないでしょう？

##一般的な解決策

一般的な解決法としては、すべての透明な三角形をソートするというものがあります。そうです、すべての透明な三角形です。

* 隠れてしまう透明な三角形を排除するために、まずは不透明な部分を描画します。
* 遠くのものから近くのものへ、三角形をソートします。
* 透明な三角形を描画します。

C言語のqsortやC++のstd::sortなど好きな方法でソートできます。これらの詳細については深入りしません。

##警告

上で述べた方法は確かに機能します。しかし：

* 描画速度が制限されます。つまり各フラグメントが10回、20回あるいはそれ以上描画されます。通常深度バッファは遠くのフラグメントを除外することを許しています。しかしここでは、明示的にそれらをソートし、深度バッファを事実上使っていません。
* もっと良い方法を使わない限り、一つのピクセルに対してこの処理を4回行うことになります。（4xMSAAを使っているため。）
* すべての透過三角形をソートするには時間がかかります。
* 三角形から三角形へテクスチャやシェーダを切り替える必要があるならば、パフォーマンスに大きな影響が出るでしょう。その場合はこの方法を使わないようにしましょう。

必要十分な解決策：

* 透過ポリゴンの最大数を制限する。
* 同じシェーダや同じテクスチャを使用する。
* まるで違ったように見えるなら、あなたのテクスチャを使いましょう。
* ソートを避けても、*あまり*悪くならないなら、それは単に運が良いだけだと思いましょう。


##順不同透過

どうしても最新の透過技術が必要なら、他のテクニックを探してみるのも良いでしょう：

* [The original 2001 Depth Peeling paper](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.18.9286&rep=rep1&type=pdf): ピクセルパーフェクト、あまり速くはない。
* [Dual Depth Peeling](http://developer.download.nvidia.com/SDK/10/opengl/src/dual_depth_peeling/doc/DualDepthPeeling.pdf) : ちょっとした改善が見られる。
* バケットソートに関する論文。フラグメントの配列を使用して、シェーダで深度によりソートする。
* [ATI's Mecha Demo](http://fr.slideshare.net/hgruen/oit-and-indirect-illumination-using-dx11-linked-lists) : とても良い。しかし改善が難しく、最新のハードウェアが必要です。フラグメントのリンクリストを使用します。
* [Cyril Crassin's variation on the ATI's  technique](http://blog.icare3d.org/2010/07/opengl-40-abuffer-v20-linked-lists-of.html) : 実装が難しい。

ハイスペックの端末で動かすリトルビッグプラネットのような最新のゲームでさえ、たった透過に1レイヤーしか使われていません。

#ブレンド関数

上のコードのために、ブレンド関数をセットアップする必要があります。
{% highlight cpp linenos %}
// ブレンドの有効化
glEnable(GL_BLEND);
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
{% endhighlight %}
これは次のことを意味します。
{% highlight text linenos %}
フレームバッファ内の新たな色 =
           フレームバッファの現在のアルファ * フレームバッファ内の現在の色 +
           (1 - フレームバッファ内の現在のアルファ) * シェーダの出力色
{% endhighlight %}
赤を上にした上で示した画像の例：
{% highlight text linenos %}
new color = 0.5*(0,1,0) + (1-0.5)*(1,0.5,0.5); // (赤は既に白い背景とブレンドされています。)
new color = (1, 0.75, 0.25) = オレンジと同じです
{% endhighlight %}
 

 
