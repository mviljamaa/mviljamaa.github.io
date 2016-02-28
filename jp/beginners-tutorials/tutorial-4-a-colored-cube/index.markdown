---
layout: page
status: publish
published: true
title: チュートリアル4：色つきの立方体
date: '2014-07-30 02:59:12 +0100'
date_gmt: '2014-07-30 02:59:12 +0100'
categories: []
tags: []
comments: []
language: jp
order: 40
---

4回目のチュートリアルにようこそ！ここでは次のことを行います。

* 単純な三角形ではなく、立方体を描きます。
* ファンシーな色を塗ります。
* Zバッファとは何かを学びます。


#立方体の描画

立方体は6枚の正方形から構成されています。OpenGLは三角形しか認識しないので、12枚の三角形を描く必要があります。つまり各面に2枚の三角形です。三角形でしたように、頂点を定義します。
{% highlight cpp linenos %}
// 頂点。3つの連続する数字は3次元の頂点です。3つの連続する頂点は三角形を意味します。
// 立方体は、それぞれが2枚の三角形からなる6面から成ります。だから6*2=12枚の三角形と12*3個の頂点を作ります。
static const GLfloat g_vertex_buffer_data[] = {
    -1.0f,-1.0f,-1.0f, // 三角形1:開始
    -1.0f,-1.0f, 1.0f,
    -1.0f, 1.0f, 1.0f, // 三角形1:終了
    1.0f, 1.0f,-1.0f, // 三角形2:開始
    -1.0f,-1.0f,-1.0f,
    -1.0f, 1.0f,-1.0f, // 三角形2:終了
    1.0f,-1.0f, 1.0f,
    -1.0f,-1.0f,-1.0f,
    1.0f,-1.0f,-1.0f,
    1.0f, 1.0f,-1.0f,
    1.0f,-1.0f,-1.0f,
    -1.0f,-1.0f,-1.0f,
    -1.0f,-1.0f,-1.0f,
    -1.0f, 1.0f, 1.0f,
    -1.0f, 1.0f,-1.0f,
    1.0f,-1.0f, 1.0f,
    -1.0f,-1.0f, 1.0f,
    -1.0f,-1.0f,-1.0f,
    -1.0f, 1.0f, 1.0f,
    -1.0f,-1.0f, 1.0f,
    1.0f,-1.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
    1.0f,-1.0f,-1.0f,
    1.0f, 1.0f,-1.0f,
    1.0f,-1.0f,-1.0f,
    1.0f, 1.0f, 1.0f,
    1.0f,-1.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
    1.0f, 1.0f,-1.0f,
    -1.0f, 1.0f,-1.0f,
    1.0f, 1.0f, 1.0f,
    -1.0f, 1.0f,-1.0f,
    -1.0f, 1.0f, 1.0f,
    1.0f, 1.0f, 1.0f,
    -1.0f, 1.0f, 1.0f,
    1.0f,-1.0f, 1.0f
};
{% endhighlight %}
OpenGLバッファは、標準の関数によって作成され、バインドされ、満たされ、設定されます。(glGenBuffers, glBindBuffer, glBufferData, glVertexAttribPointer):記憶が怪しければチュートリアル2を見てください。描画コールはどれも変更しません。あなたはただ描画したい頂点の正しい数値をセットする必要があるだけです。
{% highlight cpp linenos %}
// 三角形の描画！
glDrawArrays(GL_TRIANGLES, 0, 12*3); // 12*3頂点は0から始まる -> 12枚の三角形 -> 6枚の正方形
{% endhighlight %}
このコードについて少し説明します。

* ここでは固定した3Dモデルを扱います。モデルを変更するためには、ソースコードを変更し、再コンパイルし、うまくいくことを望むことです。チュートリアル7では動的にモデルをロードする方法を学びます。
* 各頂点は少なくとも3回書く必要があります。(上のコードで"-1.0f,-1.0f,-1.0f"を検索してみてください)。これはメモリの無駄使いです。チュートリアル9ではこれに対処する方法を学びます。

これで真っ白な三角形を描く準備が整いました。シェーダを動かしましょう！トライしてみましょう:)

#色の追加

色は、概念的には、位置とまったく同じです。つまりただのデータです。OpenGLの用語でいえば"属性"です。実は、glEnableVertexAttribArray() と glVertexAttribPointer()で、既に使っています。他の属性も追加しましょう。コードは同じように書けます。

最初に、色を宣言します。各頂点に3つ組のRGBが一つあります。ここでは、私がランダムに作りました。だから見栄えはあまりよくありません。しかし、あなたはよりよくすることができるでしょう。例えば、頂点の位置をそれ自身の色にコピーすることでも実現できるでしょう。
{% highlight cpp linenos %}
// 各頂点に一つの色。これらはランダムに作られました。
static const GLfloat g_color_buffer_data[] = {
    0.583f,  0.771f,  0.014f,
    0.609f,  0.115f,  0.436f,
    0.327f,  0.483f,  0.844f,
    0.822f,  0.569f,  0.201f,
    0.435f,  0.602f,  0.223f,
    0.310f,  0.747f,  0.185f,
    0.597f,  0.770f,  0.761f,
    0.559f,  0.436f,  0.730f,
    0.359f,  0.583f,  0.152f,
    0.483f,  0.596f,  0.789f,
    0.559f,  0.861f,  0.639f,
    0.195f,  0.548f,  0.859f,
    0.014f,  0.184f,  0.576f,
    0.771f,  0.328f,  0.970f,
    0.406f,  0.615f,  0.116f,
    0.676f,  0.977f,  0.133f,
    0.971f,  0.572f,  0.833f,
    0.140f,  0.616f,  0.489f,
    0.997f,  0.513f,  0.064f,
    0.945f,  0.719f,  0.592f,
    0.543f,  0.021f,  0.978f,
    0.279f,  0.317f,  0.505f,
    0.167f,  0.620f,  0.077f,
    0.347f,  0.857f,  0.137f,
    0.055f,  0.953f,  0.042f,
    0.714f,  0.505f,  0.345f,
    0.783f,  0.290f,  0.734f,
    0.722f,  0.645f,  0.174f,
    0.302f,  0.455f,  0.848f,
    0.225f,  0.587f,  0.040f,
    0.517f,  0.713f,  0.338f,
    0.053f,  0.959f,  0.120f,
    0.393f,  0.621f,  0.362f,
    0.673f,  0.211f,  0.457f,
    0.820f,  0.883f,  0.371f,
    0.982f,  0.099f,  0.879f
};
{% endhighlight %}
バッファは前とまったく同じ方法で、作成され、バインドされ、満たされます。
{% highlight cpp linenos %}
GLuint colorbuffer;
glGenBuffers(1, &colorbuffer);
glBindBuffer(GL_ARRAY_BUFFER, colorbuffer);
glBufferData(GL_ARRAY_BUFFER, sizeof(g_color_buffer_data), g_color_buffer_data, GL_STATIC_DRAW);
{% endhighlight %}
設定も同様です。
{% highlight cpp linenos %}
// 2nd attribute buffer : colors
glEnableVertexAttribArray(1);
glBindBuffer(GL_ARRAY_BUFFER, colorbuffer);
glVertexAttribPointer(
    1,                                // 属性。1という数字に意味はありません。ただしシェーダのlayoutとあわせる必要があります。
    3,                                // サイズ
    GL_FLOAT,                         // タイプ
    GL_FALSE,                         // 正規化？
    0,                                // ストライド
    (void*)0                          // 配列バッファオフセット
);
{% endhighlight %}
ここで、頂点バッファ内で、追加のバッファへのアクセスするために次のものを書きます。
{% highlight glsl linenos cssclass=highlightglslvs %}
// ここでの "1" はglVertexAttribPointerの "1" と同じにします。
layout(location = 1) in vec3 vertexColor;
{% endhighlight %}
今回は、頂点シェーダではファンシーにするための作業は何もしません。ただフラグメントシェーダへ送るだけです。
{% highlight glsl linenos cssclass=highlightglslvs %}
// アウトプットデータ。各頂点に書き込まれます。
out vec3 fragmentColor;

void main(){

    [...]

    // 各フラグメントの色を作るために
    // 各頂点の色は書き込まれます。
    fragmentColor = vertexColor;
}
{% endhighlight %}
フラグメントシェーダでは再びfragmentColorを宣言します。
{% highlight glsl linenos cssclass=highlightglslfs %}
// 頂点シェーダから書き込まれた色
in vec3 fragmentColor;
{% endhighlight %}
...そして、最終的なアウトプットカラーにコピーします。
{% highlight glsl linenos cssclass=highlightglslfs %}
// アウトプットデータ
out vec3 color;

void main(){
    // アウトプットカラー = 頂点シェーダで指定された、
    // 周辺の3つの頂点間で書き込まれた色
    color = fragmentColor;
}
{% endhighlight %}
これが出力結果です。

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/missing_z_buffer.png)


なんともひどい出来です。この現象を理解するために、"遠く"の三角形と"近く"の三角形を描いたときに何が起こるかを見ましょう。

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/FarNear.png)


OKでしょう。では、"遠く"の三角形を最後に書いた場合はどうでしょう。

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/NearFar.png)


"近く"の三角形に上書きされます。たとえその後ろにあると想定していてもです。これが私たちの立方体でも起きています。隠れると想定されている面がありますが、最後に描画されたために見えてしまっています。Zバッファに助けを求めましょう！

*メモ１* : 問題がないように見えるなら、カメラの位置を(4,3,-3)に変更してください。

*メモ2* : "色は位置のように属性だ"というのなら、なぜ位置ではしなかったout vec3 fragmentColorとin vec3 fragmentColorを宣言する必要があるのでしょうか？なぜなら、実は位置は少し特別だからです。位置は唯一強制されるものだからです。(もし位置を宣言しなければ、OpenGLはどこに三角形を描けばいいのか分かりません！)だから頂点シェーダでは、gl_Positionは"組み込み"の変数です。

#Zバッファ

この問題を解決するには、バッファ内の各フラグメントのデプス(例えば"Z")要素を保存することです。そして描画するごとに、まず描画すべきかどうかをチェックします。(例えば、新しいフラグメントは以前のものよりも近いかどうか。)

自分自身でも出来ますが、ハードウェア自身にその作業をさせるほうがよりシンプルです。
{% highlight cpp linenos %}
// デプステストを有効にする
glEnable(GL_DEPTH_TEST);
// 前のものよりもカメラに近ければ、フラグメントを受け入れる
glDepthFunc(GL_LESS);
{% endhighlight %}
色だけでなくデプスも毎フレームクリアする必要があります。
{% highlight cpp linenos %}
// スクリーンをクリアする
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
{% endhighlight %}
この問題を解決するにはこれで充分です。

![]({{site.baseurl}}/assets/images/tuto-4-colored-cube/one_color_per_vertex.png)


#演習


* 立方体と三角形を異なった位置に描画しましょう。そのためには、2つのMVP行列を作り、メインループで2回描画コールするでしょう。しかしシェーダは一つだけのはずです。


* 自分自身で色の値を作りましょう。例：実行ごとに変わるようにランダムにする、頂点の位置によって変わるようにする、その2つを合わせるなどです。他にもクリエイティブなアイディアがあるでしょう。 :) C言語を知らない人のために、文法を示しておきます。

{% highlight cpp linenos %}
static GLfloat g_color_buffer_data[12*3*3];
for (int v = 0; v < 12*3 ; v++){
    g_color_buffer_data[3*v+0] = ここに赤色
    g_color_buffer_data[3*v+1] = ここに緑色
    g_color_buffer_data[3*v+2] = ここに青色
}
{% endhighlight %}

* それが出来たら、フレームごとに色を変えてみましょう。glBufferDataを毎フレーム呼ぶ必要があるでしょう。適切なバッファをバインド(glBindBuffer)したかを確かめましょう！

