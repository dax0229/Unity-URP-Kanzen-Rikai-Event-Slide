---
# テーマ一覧 : https://sli.dev/themes/gallery.html
theme: seriph
# theme: default
# theme: apple-basic
# theme: bricks
# theme: unicorn

presenter: dev

# シンタックスハイライト 'prism'か'shiki'が選択可能です。
# highlighter: 'prism'
# highlighter: 'prism'
highlighter: 'shiki'

# スライドのカラースキーマを変更します。'auto'、'light'または'dark'を指定可能です。
colorSchema: 'dark'

# transition: slide-left
# transition: fade

# レイアウト : https://ja.sli.dev/builtin/layouts.html#iframe-right
# layout: center # コンテンツを画面中央に表示します。
# layout: default # 最も基本的なレイアウトで、あらゆる種類のコンテンツを表示することができます。
# layout: end
# layout: fact
# layout: full
# layout: image-left
# layout: intro
# layout: quote # 引用文を目立つように表示します。
# layout: section # 新しいプレゼンテーションのセクションの開始を示すために使用します。
# layout: center
# layout: none # スタイルなしのレイアウトです。
# layout: statement # 断言/宣言をメインページのコンテンツとして表示します。

lineNumbers: true
layout: intro # プレゼンテーションの始まりに使用します。一般的にはプレゼンテーションのタイトル、簡潔な説明、著者などです。

# "PT Mono",ui-monospace,SFMono-Regular,Menlo,Monaco,Consolas,"Liberation Mono","Courier New",monospace
  
---

<h style="opacity:100% ; margin-bottom:-8px">
Unity URP 完全に理解した勉強会 
</h> 
<h2 style="margin-top:2px ; margin-bottom:28px">
<span>URP</span>
<span style="font-size:88% ; padding-left:3px;">の</span>
<span style="font-size:92%">シェーダー</span>
<span style="font-size:92% ; top:0px ; position:relative;">の</span>
<span style="font-size:95%">書き方</span>
<span style="font-size:93% ; position:relative; top:2px">を</span>
<span style="font-size:98%">理解</span>
<span style="font-size:92%">しよう</span>
</h2>
<h4 style="opacity:58% ; margin-top:-5px;">2024年 2月 2日</h4>

---

## 自己紹介 

|||
|:---|:---|
|Twitter|かもそば(@rn49rn49)|
|所属|株式会社 Colorful Palette|
|職種|クライアントエンジニア / グラフィックスエンジニア|
|業務| インゲームの機能開発 / 描画周りの開発  など |

<br>

<img src ="/data/twitter_icon.png" style="width:120px ; position:relative; left: 16px">

<!--
19
-->

---
hideInToc: true
---

## 今回の内容

【内容】
- 前半 : <span class="highlight">ShaderLab</span> と <span class="highlight">ShaderGraph</span> の比較
- 後半 : <span class="highlight">ビルトインRP</span>と<span class="highlight">URP</span>におけるシェーダー(ShaderLab)の<span class="highlight">書き方の違い</span>

<br>

<h4>【環境】</h4>
Unity 2022.3.18f1 , Universal RP 14.0.10<br>
<br>

<h4>【対象者】</h4>
URPのシェーダーの書き方を理解していない人<br>

<br>

<h4>【注意事項】</h4> 

<div> ※ XR のシェーダーの話はしません</div>

<!--
40
-->

---
layout: section
---

<h4>Chapter 1</h4>
<h2 class="section-title"> ShaderLab と ShaderGraph</h2>

<!--
54
-->

---

## シェーダーを作成する2つの手法

1. ShaderLab
    - <u>コード記述</u>によるシェーダー作成 <br>

<br>
<br>

2. ShaderGraph
    - <u>ノードベース</u>によるシェーダー作成 <br>


<br>

<!--
Unityでシェーダーを作成するには二つの方法があります
-->

---

## ShaderLab
コードを記述してシェーダーを作成。エンジニア向き
<Transform scale=90%>
```hlsl{none}
Varyings vert (Attributes v)
{
    Varyings o = (Varyings)0;
    const VertexPositionInputs positionInputs = GetVertexPositionInputs(v.positionOS);
    o.positionCS = positionInputs.positionCS;
    o.positionWS = positionInputs.positionWS;
    o.normalWS = TransformObjectToWorldNormal(v.normalOS);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}
half4 frag (Varyings i) : SV_Target
{
    // テクスチャから色を取得 (テクスチャサンプリング)
    half4 color = tex2D(_MainTex, i.uv); 
    // フレネルを計算
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(i.positionWS);
    half dotNV = dot(i.normalWS, viewDirWS);
    half fresnel = pow(1.0 - dotNV, 2.0);
    // フレネルを合成
    half t = fresnel + color.r;
    color.rgb = lerp(_Color1.rgb, _Color2.rgb, t);
    return color;
}
```
</Transform>

<img src ="/data/images/sample_look.drawio.png" style="position:absolute; right:30px ; bottom:12% ; width:32% ">

<!--
古くからあるものなので馴染み深い
-->

---

## ShaderGraph
ノードベースでシェーダーを作成。視覚的に処理を組めるのでアーティストも触れる <br>
(ShaderGraphは、Unity2021.2からビルトインRPでも利用可能に)

<img src ="/data/images/sample_shadergraph.drawio.png" style="width:72%">
<img src ="/data/images/sample_look.drawio.png" style="position:absolute; right:30px ; bottom:12% ; width:32% ">

---

## ShadrerLabのメリット : コードレビューのしやすさ (1/2)

ShaderLabは<b>テキスト形式</b>なため、<b>Gitで差分</b>が確認できる(<b class>コードレビューがやりやすい</b>)

<img src ="/data/images/shaderlab_git_diff.drawio.png">


---

## ShadrerLabのメリット : コードレビューのしやすさ (2/2)
ShaderGraphは<b class="warn">YAML形式</b>なため、Gitで差分が追いにくい(<b class="warn">コードレビューがやりにくい</b>)<br>
(ShaderGraphの差分が確認できるツールがあれば...)

<img src ="/data/images/shadergraph_git_diff.drawio.png" style="height:70%">

---

## ShaderGraphのメリット : 効率の良いシェーダーになる (1/3)

ShaderGraphでシェーダーを作成すると、以下のようなメリットがある
- <b>SRP Batcher</b> 対応のシェーダーになる
- シェーダーアセンブリの<b>命令数を少なくしてくれる</b>

-> シェーダーへの深い知識が無くても、ある程度<b>効率の良い</b>シェーダーになる


---

## ShaderGraphのメリット : 効率の良いシェーダーになる (2/3)

ShaderLabでシェーダーを書いた場合、<b>アセンブリの命令数が多くなる</b>ケースがある <br>
<br>

<div class="grid grid-cols-2 gap-2"> <!-- Top -->

<div> <!-- Left Top -->

<h4>パフォーマンスの<span class="warn">悪い</span>処理</h4>
命令数が<span class="warn">2つ</span>になるシェーダー

<Transform>
```hlsl{none}
half4 frag()
{
    return (_A + 3.0f) * 0.5f;
}
```
</Transform>
</div> <!-- End Left Top -->

<div> <!-- Right Top -->

<h4>パフォーマンスの<span class="safe">良い</span>処理</h4>
命令数が<span class="safe">1つ</span>になるシェーダー

<Transform>
```hlsl{none}
half4 frag()
{
    return _A * 0.5f + 1.5f;
}
```
</Transform>

</div> <!-- End Right Top -->

</div> <!-- End Top -->
 
<br>

<div class="grid grid-cols-2 gap-2"> <!-- Bottom -->

<div> <!-- Bottom Left -->
命令の数: <b class="warn">2</b> (<u>add</u>  と <u>mul</u>)

<Transform scale=84% style="width:119%">
```asm{none}
    ps_4_0
    dcl_constantbuffer CB0[128], immediateIndexed
    dcl_output o0.xyzw
    dcl_temps 1
0: add r0.x, cb0[127].x, l(3.0)
1: mul o0.xyzw, r0.xxxx, l(0.5, 0.5, 0.5, 0.5)
2: ret 
```
</Transform>

</div> <!-- End Bottom Left -->


<div> <!-- Bottom Right -->
命令の数:<b class="safe">1</b> (<u>mad</u>のみ)
<Transform scale=84% style="width:128%">
```asm{none}
    ps_4_0
    dcl_constantbuffer CB0[128], immediateIndexed
    dcl_output o0.xyzw
0: mad o0.xyzw, cb0[127].xxxx, l(0.5, 0.5, 0.5, 0.5), l(1.5, 1.5, 1.5, 1.5)
1: ret 
```
</Transform>

</div> <!-- End Bottom Right -->

</div> <!-- EndBottom -->

---

## ShaderGraphのメリット : 効率の良いシェーダーになる (3/3)

ShaderGraphで似た処理を書いた場合、アセンブリの<b>命令が１つ</b>になる

<img src ="/data/images/shadergraph_mad.drawio.png">

アセンブリを見てみると、どちらのShaderGraphも<u>mad</u>命令が<b>1つだけ</b>になる <br>

```asm{none}
ps_4_0
   dcl_constantbuffer CB0[1], immediateIndexed
   dcl_output o0.xyzw
0: mad o0.xyz, cb0[0].xxxx, l(0.5, 0.5, 0.5, 0.0), l(1.5, 1.5, 1.5, 0.0)
1: mov o0.w, l(1.0)
2: ret
```

---

## ShaderGraphのデメリット : 一部の機能を使用できない

標準のShaderGraphは、<b>一部の機能を使用できない</b><br>

1. <b>ステンシル</b>が使えない

```hlsl {none}
Stencil
{
    Ref 1
    Comp Always
    Pass Replace
}
```

2. <b>複数パス対応のシェーダー</b>が作れない
```hlsl{none}
SubShader
{
    Pass { vert(){...} frag() {...} }
    Pass { vert(){...} frag() {...} }
}
```

※ ただし、ShaderGraphを改造することで、これらの問題は解決 (ShaderGraphのTargetを使用)<br>
参考 : https://qiita.com/suzuna-honda/items/362dd5b919583130c3ec#shadergraph%E3%82%92%E6%8B%A1%E5%BC%B5%E3%81%99%E3%82%8B

---

## まとめ : ShaderLab と ShaderGraph の比較

||ShaderLab|ShaderGraph|
|:---|:---:|:---:|
|シェーダー作成の方法|コーディング|ノードベース|
|シェーダー作成の難易度|エンジニア向き | アーティスト向き |
|処理の最適化|<span>自前で最適化する</span>|<b class="safe">〇効率の良いシェーダーにしてくれる</b>|
|レビューのしやすさ|<span class="safe">◎ : しやすい</span> | <span class="warn">△ : 難しい</span> |
|イテレーションの回しやすさ|<span class="warn">△ : 遅い</span>|<span class="safe">◎ : 速い</span>|
|機能|<span class="safe">◎ : すべての機能が使える</span>|<b>△ : 機能が制限されている</b>|

---
layout: section
---

<h4>Chapter 2</h4>
<h2 class="section-title"> URP向けのシェーダー(ShaderLab)の書き方</h2>

<!--
4 : 00
-->

---

## ビルトインRPとURPの違い

### ビルトインRP
<ul>
<li v-click="1">基本的に<b>組み込み</b>のレンダーパイプライン上でシェーダーを実装</li>
<li v-after>描画の<b class="warn">拡張の自由度は低い</b>が、<b class="safe">簡単に扱うことができる</b></li>
<li v-after><b class="safe">ライティングを簡単に構築</b>できる<b>Surfaceシェーダー</b>が用意されている</li>
</ul>

<span v-after>-> グラフィックスの実装に<b>コストをかけたくない場合</b>、ビルトインRPがおすすめ</span>

<br>

### URP

<ul>
<li v-click="2">レンダーパイプラインを<b>拡張</b>できる仕組みが用意されている (<u>ScriptableRendererFeature</u>など)</li>
<li v-after>描画の<b class="safe">拡張の自由度は高い</b>が、拡張には3Dグラフィックスへの<b class="warn">深い知識が必要</b>となる</li>
<li v-after><b>Surfaceシェーダー</b>は<b class="warn">使用不可</b> (<b>ライティング</b>は自分で実装)</li>
</ul>

<span v-after><b>-> グラフィックスにこだわりたい</b>場合、URPがおすすめ (ビルトインRPより負荷が軽量)</span>

<!--
シェーダーに入る前に、ビルトインRPとURPの違いについておさらいしてみましょう。
-->

---
clicks: 5
---

## ビルトインRPとURPのシェーダーの違い

<b>URP</b>の シェーダー(ShaderLab)の書き方は、<b>ビルトインRP</b>のものと大きく異なる


<br>

<div class="grid grid-cols-2" style="font-size:92%">

<div>

<h3>ビルトイン RP</h3>

<ol>
<li v-click="1">シェーダーは<b>Cg言語</b>で記述する<br></li>
<li v-click="2">構造体の名前は <b>appdata</b> や <b>v2f</b> にする</li>
<li v-click="3">頂点変換には <b>UnityObjectToClipPos</b>関数を使う </li>
<li v-click="4">テクスチャの定義に <b>sampler2D</b>関数を使う</li>
<li v-click="5">テクスチャサンプリングには <b>tex2D</b>関数を使う</li>
</ol>

</div>

<div class="position-relative right-15px size-700px">

<h3>URP</h3>
<!-- <ol v-click="2"> -->

<ol>
<li v-click="1">シェーダーは<b>HLSL</b>で記述する</li>
<li v-click="2">構造体の名前は <b>Varyings</b> や <b>Attributes</b> にする</li>
<li v-click="3">頂点変換には <b>TransformObjectToHClip</b> を使う </li>
<li v-click="4">テクスチャの定義に <b>TEXTURE2D</b> や<b>SAMPLER</b>を使う</li>
<li v-click="5">テクスチャサンプリングには <b>SAMPLE_TEXTURE2D</b> を使う</li>
</ol>


</div>

</div>


---

## 言語の違い (1/2)
URP では、シェーディング言語に<b>HLSL</b>の使用が推奨されます。<br>
(URP内のシェーダーはHLSLを使って記述されています)

<br>

<!-- Grid -->
<div class="grid grid-cols-2">

<!-- ビルトイン RP -->
<div>
<h3>Cg言語</h3>

<code class="highlight">CGPROGRAM</code> と <code class="highlight">ENDCG</code>で囲む

<!-- code-->
<!-- <div class="code-highlight"> -->
<div>
<Transform scale=120% style="left:0px ; width : 320px">

```hlsl{1,7}
CGPROGRAM
#pragma fragment frag
#pragma vertex vert

(省略)

ENDCG
```
</Transform>

</div>
<!-- End code-->

</div>
<!-- End ビルトイン RP -->

<!-- URP -->
<div style="width:660px ; position:relative ; width : 600px">
<!-- <div> -->
<h3>HLSL</h3>

<code class="highlight">HLSLPROGRAM</code> と <code class="highlight">ENDHLSL</code>で囲む

<!-- code -->
<!-- <div class="code-highlight" style="width:500px"> -->
<div>
<Transform scale=120%>

```hlsl{1,7}
HLSLPROGRAM
#pragma fragment frag
#pragma vertex vert
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
(省略)

ENDHLSL

```
</Transform>

</div>
<!-- End code-->

</div>
<!-- End URP -->

</div>


---

## 言語の違い (2/2)


<div class="grid grid-cols-1 gap-7">

<div>

<h4><u>CGPROGRAM</u>で書いた場合</h4>
<div v-click="1">
シェーダーライブラリが<b>自動でインクルードされます</b>

> HLSLSupport.cginc, UnityShaderVariables.cginc, AutoLight.cginc , Lighting.cginc , TerrainEngine.cginc 


シェーダーライブラリは<b>UnityEditorに内蔵</b>されているため、シェーダーライブラリの<b class="warn">改造は難しい</b>です<br>

シェーダーライブラリの場所 : <i>{Unityエディターのインストールフォルダ}/Editor/Data/CGIncludes</i><br> 
</div>

</div>
<div>
<h4><u>HLSLPROGRAM</u>で書いた場合 (推奨)</h4>
<div v-click="2">
シェーダーライブラリが自動インクルードされないため、明示的に<code class="highlight">#include</code>を書く必要があります。
<Transform scale=100%>

```hlsl{none}
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
```
</Transform>


シェーダーライブラリは<b>Packages内に格納</b>されているため、シェーダーライブラリの<b>改造</b>や・プロジェクト内<b>共有</b>が<b class="safe">簡単に行えます</b>

URPの長所でもある、<u>拡張性の高さ</u>を生かすなら <u>HLSLPROGRAM</u>で書くのが良いでしょう

</div>

</div>

</div>

---

## 構造体 (struct)

ビルトインRPでは、構造体の名前を <code class="highlight">appdata</code>や<code class="highlight">v2f</code> と書いていました<br>

URPでは、構造体の名前を <code class="highlight">Attributes</code>や<code class="highlight">Varyings</code> にすることが推奨されます<br>

構造体の中の変数には<code class="highlight">positionOS</code> や <code class="highlight">positionCS</code> のように、空間をつけるとURPらしいコードに<br>

<br>

<!-- Grid -->
<div class="grid grid-cols-2 gap-2">

<!-- ビルトイン RP -->
<div>
<h3>ビルトイン RP</h3>

<div class="code-highlight">
<Transform scale=85%>

```hlsl{none}
struct appdata
{
    float4 vertex : POSITION; // 頂点座標
    float2 uv : TEXCOORD0; // 法線
};

struct v2f
{
    float4 vertex : SV_POSITION; // 頂点座標
    float2 uv : TEXCOORD1; // テクスチャ座標
};
```

</Transform>
</div>

</div>
<!-- End ビルトイン RP -->

<!-- URP -->
<div>

<h3>URP</h3>

<!-- code-highlight -->
<div class="code-highlight">
<Transform scale=90%>

```hlsl{none}
struct Attributes
{
    float4 positionOS : POSITION; // 頂点座標(Object Space)
    float2 uv : TEXCOORD0;
};

struct Varyings
{
    float4 positionCS : SV_POSITION; // 頂点座標(Clip Space)
    float2 uv : TEXCOORD1;
};
```

</Transform>
</div>
<!-- End code-highlight -->

</div>
<!-- End URP -->

</div>
<!-- End Grid -->


---

## 頂点処理 (vertex shader)
URPでは、頂点シェーダーで使用する<b>頂点変換系の関数が変わる</b>

- Cg言語では、<code class="highlight">UnityObjectToClipPos</code> を使用し座標変換を行う<br>
- HLSLでは、<code class="highlight">TransformObjectToHClip</code> を使用して座標変換を行う <br>
<span><Transform scale=65% class="path">マクロの定義場所 : Packages/com.unity.render-pipelines.universal/ShaderLibrary/SpaceTransforms.hlsl</Transform></span>

<br>
<!-- Grid -->
<div class="grid grid-cols-2 gap-4" style="font-size:92%">
<!-- <div class="grid grid-cols-1 gap-0"> -->
<!-- <div class="grid grid-cols-1" style="font-size:92%"> -->

<!-- ビルトイン RP -->
<div>
<h3>ビルトイン RP</h3>

<!-- code-highlight -->
<!-- <div class="code-highlight" style="width:480px"> -->
<div class="code-highlight">
<!-- <div class="code-highlight" style=""> -->
<Transform scale=95%>
```hlsl{4}
v2f vert (appdata v)
{
    v2f o;
    o.vertex = UnityObjectToClipPos(v.vertex.xyz);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}
```
</Transform>

</div>
<!-- End code-highlight -->

</div>
<!-- End ビルトイン RP -->

<!-- URP -->
<!-- <div class="code-highlight"> -->
<div style="width:510px ; position:relative">
<!-- <div style="position:relative ; right: 0px"> -->

<h3>URP</h3>

<!-- code-highlight -->
<!-- <div class="code-highlight"> -->
<div class="code-highlight">
<Transform scale=95%>

```hlsl{4}
Varyings vert (Attributes v)
{
    Varyings output = (Varyings)0; // 0で初期化
    o.positionCS = TransformObjectToHClip(v.positionOS.xyz);
    o.uv = TRANSFORM_TEX(v.uv, _MainTex);
    return o;
}
```

</Transform>
</div>
<!-- End code-highlight -->

</div>
<!-- End URP -->

</div>
<!-- End Grid -->


Core.hlslをインクルードすると使用可能

```hlsl{none}
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
```


---

## 頂点変換系の関数

URPには座標変換系の関数が多く用意されています。

<Transform scale=130%>
```hlsl {none}
float3 TransformObjectToWorld(float3 positionOS)
float3 TransformWorldToObject(float3 positionWS)
float3 TransformWorldToView(float3 positionWS)
float4 TransformObjectToHClip(float3 positionOS)
float4 TransformWorldToHClip(float3 positionWS)
float4 TransformWViewToHClip(float3 positionVS)
float3 TransformTangentToWorld(float3 dirTS, float3x3 tangentToWorld)
float3 TransformWorldToTangent(float3 dirWS, float3x3 tangentToWorld)
float3 TransformTangentToObject(float3 dirTS, float3x3 tangentToWorld)
float3 TransformObjectToTangent(float3 dirOS, float3x3 tangentToWorld)
```
</Transform>

<br>
<br>
<br>
Core.hlslをインクルードすると使用可能


---

## 便利な関数 : GetVertexPositionInputs

頂点変換をまとめて行ってくれる便利な関数<code class="highlight">GetVertexPositionInputs</code>があります。<br>
<span><Transform scale=65% class="path">マクロの定義場所 : Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl</Transform></span>

```hlsl{none}
// 頂点座標の変換
VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
float3 positionWS = vertexInput.positionWS; // ワールド座標を取得
float3 positionVS = vertexInput.positionVS; // ビュー空間の座標を取得
float3 positionCS = vertexInput.positionCS; // クリップ空間の座標を取得
```

Core.hlslをインクルードすると使用可能

---

## テクスチャサンプリング

テクスチャから色を取得するには、<code class="highlight">tex2D</code> を使う方法と、<code class="highlight">Texture2D.Sample</code> を使う方法があります

<div class="grid grid-cols-2 gap-4" style="">
<!--  -->

<!-- tex2Dを使う方法 -->
<div>

<h5>tex2Dを使う (DirectX シェーダーモデル 2 ~ 4 でサポート)</h5>

```glsl{none}
sampler2D _MainTex;
half4 color = tex2D(_MainTex, uv);
```
</div>
<!-- End tex2Dを使う方法 -->

<!-- Texture2D.Sampleを使う方法 -->
<div>
<h5>Texture2Dを使う (DirectX シェーダーモデル 5 以降でサポート)</h5>

```hlsl{none}
Texture2D _MainTex;
SamplerState sampler_MainTex;

half4 color = _MainTex.Sample(sampler_MainTex, uv);
```
</div>
<!-- End Texture2D.Sampleを使う方法 -->

</div>

URPで定義されているマクロは、ShaderModelによるAPIの違いを吸収してくれる。

<!-- Start Grid-->

<div class="grid grid-cols-2 gap-3" style="">

<!-- 1 -->
<div style="">
古いAPI (GLES2 など)

<!-- <Transform scale=60% style="line-height:50% ; width:700px ; padding-top:5px;">Packages/com.unity.render-pipelines.core/ShaderLibrary/API/<span class="highlight">GLES2.hlsl</span></Transform> -->
<Transform scale=60% class="path">Packages/com.unity.render-pipelines.core/ShaderLibrary/API/<span class="highlight">GLES2.hlsl</span></Transform>
<Transform scale=75% style="width:530px">
```hlsl{none}
#define TEXTURE2D(textureName) sampler2D textureName
#define SAMPLER(samplerName)
#define SAMPLE_TEXTURE2D(textureName, samplerName, coord2) \ 
tex2D(textureName, coord2)
```
</Transform>

</div>
<!-- 1 -->

<!-- 2 -->
<div style="">
新しいAPI (GLES3 / Vulkan / Metal / D3D11 など)
<!-- <Transform scale=60% style="line-height:50%; width:700px ; padding-top:5px; left:10px;">Packages/com.unity.render-pipelines.core/ShaderLibrary/API/<span class="highlight">GLES3.hlsl</span></Transform> -->
<Transform scale=60% class="path">Packages/com.unity.render-pipelines.core/ShaderLibrary/API/<span class="highlight">GLES3.hlsl</span></Transform>
<Transform scale=75% style="width:530px">
```hlsl{none}
#define TEXTURE2D(textureName) Texture2D textureName
#define SAMPLER(samplerName) SamplerState samplerName
#define SAMPLE_TEXTURE2D(textureName, samplerName, coord2) \
textureName.Sample(samplerName, coord2)
```
</Transform>
</div>
<!-- 2 -->

</div>
<!-- End Grid -->

---

## マクロを使ったテクスチャサンプリング

<div class="grid grid-cols-2 gap-4" style="">

<!--マクロを使わない書き方-->
<div>

<h5>マクロを使わない書き方</h5>

```hlsl{none}
Texture2D _MainTex;
SamplerState sampler_MainTex;
...
half4 color = 
_MainTex.Sample(sampler_MainTex, uv);
```

</div>

<!-- End マクロを使わない書き方 -->

<div>

<!-- マクロを使った書き方 -->

<h5>マクロを使った書き方</h5>

```hlsl{none}
TEXTURE2D(_MainTex);
SAMPLER(sampler_MainTex);
...
half4 color = 
SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, uv);
```
</div>

<!-- End マクロを使った書き方 -->

</div>

---

## シェーダーの比較

<!-- <arrow v-click="[1, 2]" x1="400" y1="420" x2="230" y2="330" color="#564" width="3" arrowSize="1" /> -->
<!-- <arrow v-click="[2, 3]" x1="40" y1="420" x2="30" y2="330" color="#564" width="3" arrowSize="1" /> -->

<!-- Grid -->
<div class="grid grid-cols-2 gap-4" style="font-size:92%">

<!-- Build-in RP -->
<div>
<h4>ビルトイン RP</h4>

<Transform scale=50% style="width:800px">

```hlsl {6,8,11,13,15,19,21,25,28,30,31,36,38,41}
SubShader
{
    Tags { "RenderType"="Opaque" }
    LOD 100

    Pass
    {
        CGPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "UnityCG.cginc"

        struct appdata
        {
            float4 vertex : POSITION;
            float2 uv : TEXCOORD0;
        };

        struct v2f
        {
            float4 vertex : SV_POSITION;
            float2 uv : TEXCOORD0;
        };

        sampler2D _MainTex;
        float4 _MainTex_ST;

        v2f vert (appdata v)
        {
            v2f o;
            o.vertex = UnityObjectToClipPos(v.vertex);
            o.uv = TRANSFORM_TEX(v.uv, _MainTex);
            return o;
        }

        fixed4 frag (v2f i) : SV_Target
        {
            fixed4 col = tex2D(_MainTex, i.uv);
            return col;
        }
        ENDCG
    }
}
```

</Transform>
</div>
<!-- End Build-in RP -->

<!-- URP -->
<div>
<h4>URP</h4>

<Transform scale=50% style="width:800px">

```hlsl {6,8,11,13,15,19,21,25,28,30,31,36,38,41}
SubShader
{
    Tags { "RenderType"="Opaque" }
    LOD 100

    Pass
    {
        HLSLPROGRAM
        #pragma vertex vert
        #pragma fragment frag
        #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"

        struct Attributes
        {
            float4 positionOS : POSITION;
            float2 uv : TEXCOORD0;
        };

        struct Varyings
        {
            float4 positionCS : SV_POSITION;
            float2 uv : TEXCOORD0;
        };

        sampler2D _MainTex;
        float4 _MainTex_ST;

        Varyings vert (Attributes v)
        {
            Varyings o;
            o.positionCS = TransformObjectToHClip(v.positionOS);
            o.uv = TRANSFORM_TEX(v.uv, _MainTex);
            return o;
        }

        half4 frag (Varyings i) : SV_Target
        {
            half4 col = SAMPLE_TEXTURE2D(_MainTex, sampler_MainTex, i.uv);
            return col;
        }
        ENDHLSL
    }
}
```

</Transform>
</div>
<!-- End URP -->

</div>
<!-- End Grid -->

---

## その他の違い : LightModeタグ

URP の LightModeタグは、[ビルトインRPのもの](https://docs.unity3d.com/ja/2018.4/Manual/SL-PassTags.html)と変わります。

<u>ビルトインRP</u>で使用する <code class="highlight">ForwardBase</code> や <code class="highlight">ForwardAdd</code> は
<u>URP</u> で<b class="warn">動作しなくなります</b>。<br>
代わりに、<code class="highlight">UniversalForward</code> や <code class="highlight">SRPDefaultUnlit</code> を使用します。

<div class="grid grid-cols-2 gap-4" style="font-size:92%">

<!-- Begin ビルトインRP -->
<div>

<h3>ビルトインRP</h3>

```hlsl{none}
Tags 
{
    "LightMode" = "ForwardBase" 
}
```

```hlsl{none}
Tags 
{
    "LightMode" = "ForwardAdd" 
}
```

</div>
<!-- End ビルトインRP -->


<!-- Begin UniversalRP -->
<div>

<h3>URP</h3>

```hlsl{none}
Tags 
{
    "LightMode" = "UniversalForward"
    "RenderPipeline" = "UniversalPipeline"
}
```

```hlsl{none}
Tags 
{
    "LightMode" = "SRPDefaultUnlit"
    "RenderPipeline" = "UniversalPipeline"
}
```

</div>
<!-- End UniversalRP -->

</div>

参考 : https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@14.0/manual/urp-shaders/urp-shaderlab-pass-tags.html

---

## LightModeタグの拡張

URPでは、<b>カスタムのLightModeタグ</b>を追加し、これを<b>C#から実行できる</b> (ScriptableRenderPassを使用する)

<div class="grid grid-cols-2 gap-2"> <!-- Top -->

<div>

パスタグ <code class="highlight">CustomTag</code> の定義　(シェーダー)

```hlsl{4}
Pass
{
    Name "Eyelash"
    Tags { "LightMode" = "CustomTag" }
    Blend SrcAlpha OneMinusSrcAlpha
    ZTest Always
    #pragma vertex vert
    #pragma fragment frag
}
```
</div>

<div style="width:500px">

パスタグ<code class="highlight">CustomTag</code>を指定してレンダラーを実行 (C#)

```cs{2}
var drawingSettings = RenderingUtils.CreateDrawingSettings(
        new ShaderTagId("CustomTag"), 
        ref renderingData, 
        SortingCriteria.CommonOpaque);
var filteringSettings 
    = new FilteringSettings(RenderQueueRange.all);

// レンダラーを実行(シェーダーが実行される)
context.DrawRenderers(
    renderingData.cullResults, 
    ref drawingSettings, ref filteringSettings);
```
</div>

</div>

参考: 【URP】キャラクターの透ける眉を実装してみる <br>
https://zenn.dev/r_ngtm/articles/urp-transparent-eyebrow

<!--
ScriptableRendererContextのDrawRenderers
を呼ぶと実行される

詳しくは、下記リンクのURLの記事を参照
-->

---
layout: end

---

## ご静聴ありがとうございました
