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
Unity URP 완전 정복하기 스터디 
</h> 
<h2 style="margin-top:2px ; margin-bottom:28px">
<span>URP 셰이더 작성법 이해하기</span>
<!-- <span style="font-size:88% ; padding-left:3px;">の</span>
<span style="font-size:92%">シェーダー</span>
<span style="font-size:92% ; top:0px ; position:relative;">の</span>
<span style="font-size:95%">書き方</span>
<span style="font-size:93% ; position:relative; top:2px">を</span>
<span style="font-size:98%">理解</span>
<span style="font-size:92%">しよう</span> -->
</h2>
<h4 style="opacity:58% ; margin-top:-5px;">2024년 2월 2일</h4>

---

## 자기소개 

|||
|:---|:---|
|Twitter|かもそば(@rn49rn49)|
|소속|주식회사 Colorful Palette|
|직종|클라이언트 엔지니어/ 그래픽 엔지니어|
|업무|게임 내 기능 개발 / 드로잉 관련 개발 등 |

<br>

<img src ="/data/twitter_icon.png" style="width:120px ; position:relative; left: 16px">

<!--
19
-->

---
hideInToc: true
---

## 이번 내용

【내용】
- 전반 : <span class="highlight">ShaderLab</span>과 <span class="highlight">ShaderGraph</span>의 비교
- 후반 : <span class="highlight">빌트인RP</span>와 <span class="highlight">URP</span>에서 셰이더(ShaderLab)의 <span class="highlight">제작 방법의 차이</span>

<br>

<h4>【환경】</h4>
Unity 2022.3.18f1 , Universal RP 14.0.10<br>
<br>

<h4>【대상】</h4>
URP 셰이더를 작성하는 방법을 이해하지 못하는 사람<br>

<br>

<h4>【주의사항】</h4> 

<div> ※ XR의 셰이더에 대한 이야기는 하지 않습니다.</div>

<!--
40
-->

---
layout: section
---

<h4>Chapter 1</h4>
<h2 class="section-title"> ShaderLab과 ShaderGraph</h2>

<!--
54
-->

---

## 셰이더를 만드는 두 가지 방법

1. ShaderLab
    - <u>코드 작성</u>을 통한 셰이더 제작 <br>

<br>
<br>

2. ShaderGraph
    - <u>노드 기반</u>으로 셰이더 제작<br>


<br>

<!--
Unityでシェーダーを作成するには二つの方法があります
-->

---

## ShaderLab
코드를 작성하여 셰이더를 제작합니다. 엔지니어용
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
    // 텍스처에서 색상 얻기 (텍스처 샘플링)
    half4 color = tex2D(_MainTex, i.uv); 
    // 프레넬 계산
    half3 viewDirWS = GetWorldSpaceNormalizeViewDir(i.positionWS);
    half dotNV = dot(i.normalWS, viewDirWS);
    half fresnel = pow(1.0 - dotNV, 2.0);
    // 프레넬 합성
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
노드 기반으로 셰이더를 생성합니다. 비주얼한 구성으로 아티스트도 쉽게 접근 가능 합니다.<br>
(ShaderGraph는、Unity2021.2부터 빌트인RP 에서도 사용 가능)

<img src ="/data/images/sample_shadergraph.drawio.png" style="width:72%">
<img src ="/data/images/sample_look.drawio.png" style="position:absolute; right:30px ; bottom:12% ; width:32% ">

---

## ShadrerLab의 장점: 코드 리뷰의 용이성 (1/2)

ShaderLab은 <b>텍스트 형식</b>이라서, <b>Git</b>에서 차이점 확인 가능 (<b class>코드 검토가 용이</b>)

<img src ="/data/images/shaderlab_git_diff.drawio.png">


---

## ShadrerLab의 장점: 코드 리뷰의 용이성 (2/2)
ShaderGraph는 <b class="warn">YAML 형식</b>이라서, Git에서 차이점 추적이 어려움 (<b class="warn">코드 검토가 어려움</b>)<br>
(ShaderGraph의 차이점을 확인할 수 있는 도구가 있다면...)

<img src ="/data/images/shadergraph_git_diff.drawio.png" style="height:70%">

---

## ShaderGraph의 장점 : 효율적인 셰이더가 된다 (1/3)

ShaderGraph로 셰이더를 맏늘면 다음과 같은 장점이 있다.
- <b>SRP Batcher</b> 지원 셰이더가 된다
- 셰이더 어셈블리의 <b>명령어 수를 줄여준다</b>

-> 셰이더에 대한 깊은 지식이 없어도 어느 정도 <b>효율적인</b> 셰이더가 될 수 있다


---

## ShaderGraph의 장점 : 효율적인 셰이더가 된다 (2/3)

ShaderLab에서 셰이더를 작성할 경우, <b>어셈블리 명령어 수가 많아지는 경우가 있다</b><br>
<br>

<div class="grid grid-cols-2 gap-2"> <!-- Top -->

<div> <!-- Left Top -->

<h4>성능 <span class="warn">저하</span> 처리</h4>
명령어 수 <span class="warn">2</span>개의 셰이더

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

<h4>성능 <span class="safe">좋은</span> 처리</h4>
명령어 수 <span class="safe">1</span> 개의 셰이더

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
명령어 수: <b class="warn">2</b> (<u>add</u> , <u>mul</u>)

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
명령어 수: <b class="safe">1</b> (<u>mad</u>)
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

## ShaderGraph의 장점 : 효율적인 셰이더가 된다 (3/3)

ShaderGraph에서 유사한 처리를 작성할 경우, 어셈블리의 <b>명령어</b>가 최적화 된다.

<img src ="/data/images/shadergraph_mad.drawio.png">

어셈블리를 보면 두 ShaderGraph 모두 <u>mad</u>명령어 <b>하나</b>만 존재 한다.<br>

```asm{none}
ps_4_0
   dcl_constantbuffer CB0[1], immediateIndexed
   dcl_output o0.xyzw
0: mad o0.xyz, cb0[0].xxxx, l(0.5, 0.5, 0.5, 0.0), l(1.5, 1.5, 1.5, 0.0)
1: mov o0.w, l(1.0)
2: ret
```

---

## ShaderGraph의 단점: 일부 기능 사용 불가

표준 ShaderGraph는 <b>일부 기능을 사용 할 수 없다</b><br>

1. <b>스텐실</b> 미지원

```hlsl {none}
Stencil
{
    Ref 1
    Comp Always
    Pass Replace
}
```

2. <b>다중 패스 지원 셰이더</b> 미지원
```hlsl{none}
SubShader
{
    Pass { vert(){...} frag() {...} }
    Pass { vert(){...} frag() {...} }
}
```

※ 단, ShaderGraph를 커스텀 하여, 이러한 문제는 해결 할 수 있다. (ShaderGraph의 Target사용)<br>
参考 : https://qiita.com/suzuna-honda/items/362dd5b919583130c3ec#shadergraph%E3%82%92%E6%8B%A1%E5%BC%B5%E3%81%99%E3%82%8B

---

## 요약 : ShaderLab 과 ShaderGraph 의 비교

||ShaderLab|ShaderGraph|
|:---|:---:|:---:|
|작성 방법|코드기반|노드기반|
|제작 난이도|엔지니어용 | 아티스트용 |
|최적화|<span>제작자</span>|<b class="safe">〇: 기능제공</b>|
|수정사항 확인|<span class="safe">◎ : 쉬움</span> | <span class="warn">△ : 어려움</span> |
|반복 작업|<span class="warn">△ : 느림</span>|<span class="safe">◎ : 빠름</span>|
|기능|<span class="safe">◎ : 모든기능</span>|<b>△ : 제한됨</b>|

---
layout: section
---

<h4>Chapter 2</h4>
<h2 class="section-title"> URP용 셰이더(ShaderLab)의 제작</h2>

<!--
4 : 00
-->

---

## 빌트인RP와 URP의 차이

### 빌트인RP
<ul>
<li v-click="1">기본적으로 <b>내장</b>된 렌더 파이프라인에서 셰이더를 구현</li>
<li v-after>연출의 <b class="warn">자유도는 낮지만,</b> <b class="safe">쉽게 다룰 수 있다.</b></li>
<li v-after><b class="safe">라이팅을 쉽게 구성</b>할 수 있는 <b>Surface 셰이더</b>가 제공된다</li>
</ul>

<span v-after>-> 그래픽 구현에 <b>비용을 들이고 싶지 않다면</b> 빌트인RP를 추천합니다.</span>

<br>

### URP

<ul>
<li v-click="2">렌더 파이프라인을 <b>확장</b> 할 수 있는 방식 (<u>ScriptableRendererFeature</u> 등)</li>
<li v-after>연출의 <b class="safe">자유도는 높지만</b>, 3D 그래픽에 대한 <b class="warn">깊은 지식</b>이 필요</li>
<li v-after><b>Surface 셰이더</b>는 <b class="warn">사용 불가</b> (<b>라이팅</b> 직접 구현)</li>
</ul>

<span v-after>-> <b>그래픽을 고집</b>하고 싶다면, URP를 추천 (빌트인RP보다 가벼움)</span>

<!--
シェーダーに入る前に、ビルトインRPとURPの違いについておさらいしてみましょう。
-->

---
clicks: 5
---

## 빌트인RP와 URP 셰이더 차이점

<b>URP</b>의 셰이더(ShaderLab) 작성 방식은 <b>빌트인RP</b>와 크게 다릅니다.


<br>

<div class="grid grid-cols-2" style="font-size:92%">

<div>

<h3>빌트인 RP</h3>

<ol>
<li v-click="1">셰이더는 <b>Cg Language</b>로 작성한다<br></li>
<li v-click="2">구조체 이름은 <b>appdata</b> 또는 <b>v2f</b> 로 지정</li>
<li v-click="3">정점 변환은 <b>UnityObjectToClipPos</b> 함수 사용 </li>
<li v-click="4">텍스처 정의는 <b>sampler2D</b> 함수 사용</li>
<li v-click="5">텍스처 샘플링은 <b>tex2D</b> 함수 사용</li>
</ol>

</div>

<div class="position-relative right-15px size-700px">

<h3>URP</h3>
<!-- <ol v-click="2"> -->

<ol>
<li v-click="1">셰이더는 <b>HLSL</b>로 작성한다</li>
<li v-click="2">구조체 이름은 <b>Varyings</b> 또는 <b>Attributes</b> 로 지정</li>
<li v-click="3">정점 변환은 <b>TransformObjectToHClip</b> 함수 사용 </li>
<li v-click="4">텍스처 정의는 <b>TEXTURE2D</b> 또는 <b>SAMPLER</b> 함수 사용</li>
<li v-click="5">텍스처 샘플링은 <b>SAMPLE_TEXTURE2D</b> 함수 사용</li>
</ol>


</div>

</div>


---

## 언어 차이 (1/2)
URP 에서는 세이딩 언어로 <b>HLSL</b>를 권장<br>
(URP의 셰이더는 HLSL로 작성됨)

<br>

<!-- Grid -->
<div class="grid grid-cols-2">

<!-- ビルトイン RP -->
<div>
<h3>Cg Language</h3>

<code class="highlight">CGPROGRAM</code> 과 <code class="highlight">ENDCG</code>로 감싸기

<!-- code-->
<!-- <div class="code-highlight"> -->
<div>
<Transform scale=120% style="left:0px ; width : 320px">

```hlsl{1,7}
CGPROGRAM
#pragma fragment frag
#pragma vertex vert

(생략)

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

<code class="highlight">HLSLPROGRAM</code> 과 <code class="highlight">ENDHLSL</code>로 감싸기

<!-- code -->
<!-- <div class="code-highlight" style="width:500px"> -->
<div>
<Transform scale=120%>

```hlsl{1,7}
HLSLPROGRAM
#pragma fragment frag
#pragma vertex vert
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
(생략)

ENDHLSL

```
</Transform>

</div>
<!-- End code-->

</div>
<!-- End URP -->

</div>


---

## 언어 차이 (2/2)


<div class="grid grid-cols-1 gap-7">

<div>

<h4><u>CGPROGRAM</u>으로 작성한 경우</h4>
<div v-click="1">
셰이더 라이브러리가 <b>자동으로 포함 됨</b>

> HLSLSupport.cginc, UnityShaderVariables.cginc, AutoLight.cginc , Lighting.cginc , TerrainEngine.cginc 


셰이더 라이브러리는 <b>UnityEditor에 내장</b>되어 있어서, 셰이더 라이브러리의 <b class="warn">수정이 어려움</b><br>

셰이더 라이브러리 위치 : <i>{Unity 설치 폴더}/Editor/Data/CGIncludes</i><br> 
</div>

</div>
<div>
<h4><u>HLSLPROGRAM</u>으로 작성한 경우 (권장)</h4>
<div v-click="2">
셰이더 라이브러리가 자동 포함되지 않으므로 명시적으로 <code class="highlight">#include</code> 를 작성해야 합니다.
<Transform scale=100%>

```hlsl{none}
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
```
</Transform>


셰이더 라이브러리는 <b>Packages에 지정</b>되어 있기 때문에, 셰이더 라이브러리의 <b>튜닝</b>과 프로젝트 내 <b>공유</b>가 <b class="safe">용이 합니다.</b>

URP의 장점인 <u>높은 확장성</u>을 살리려면 <u>HLSLPROGRAM</u>으로 작성하는 것이 좋습니다.

</div>

</div>

</div>

---

## 구조체 (struct)

빌트인RP 에서는 구조체 이름을 <code class="highlight">appdata</code> 또는 <code class="highlight">v2f</code> 로 작성<br>

URP에서는 구조체 이름을 <code class="highlight">Attributes</code> 또는 <code class="highlight">Varyings</code> 로 지정<br>

구조체 안에 변수에는 <code class="highlight">positionOS</code>, <code class="highlight">positionCS</code> 와 같이 공백을 붙이면 URP스러운 코드가 된다.<br>

<br>

<!-- Grid -->
<div class="grid grid-cols-2 gap-2">

<!-- ビルトイン RP -->
<div>
<h3>빌트인RP</h3>

<div class="code-highlight">
<Transform scale=85%>

```hlsl{none}
struct appdata
{
    float4 vertex : POSITION; // 정점 좌표
    float2 uv : TEXCOORD0; // 법선
};

struct v2f
{
    float4 vertex : SV_POSITION; // 정점 좌표
    float2 uv : TEXCOORD1; // 텍스처 좌표
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
    float4 positionOS : POSITION;//정점좌표(Object Space)
    float2 uv : TEXCOORD0;
};

struct Varyings
{
    float4 positionCS : SV_POSITION;//정점좌표(Clip Space)
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

## 정점 처리 (vertex shader)
URP에서는 버텍스 셰이더에서 사용하는 <b>정점 처리 함수가 다르다</b>

- Cg에서는, <code class="highlight">UnityObjectToClipPos</code> 를 사용하여 변환<br>
- HLSL에서는, <code class="highlight">TransformObjectToHClip</code> 를 사용하여 변환 <br>
<span><Transform scale=65% class="path">매크로 위치 : Packages/com.unity.render-pipelines.universal/ShaderLibrary/SpaceTransforms.hlsl</Transform></span>

<br>
<!-- Grid -->
<div class="grid grid-cols-2 gap-4" style="font-size:92%">
<!-- <div class="grid grid-cols-1 gap-0"> -->
<!-- <div class="grid grid-cols-1" style="font-size:92%"> -->

<!-- ビルトイン RP -->
<div>
<h3>빌트인 RP</h3>

<!-- code-highlight -->
<!-- <div class="code-highlight" style="width:480px"> -->
<div class="code-highlight"  style="width:420px" >
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
    Varyings output = (Varyings)0; // 초기화
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


Core.hlsl을 포함하면 사용 가능

```hlsl{none}
#include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
```


---

## 정점 변환 함수

URP에는 좌표 변환 함수가 많이 준비되어 있다.

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
Core.hlsl를 포함하면 사용 가능


---

## 유용한 함수 : GetVertexPositionInputs

정점 변환을 일괄적으로 처리해주는 함수 <code class="highlight">GetVertexPositionInputs</code><br>
<span><Transform scale=65% class="path">매크로 위치 : Packages/com.unity.render-pipelines.universal/ShaderLibrary/ShaderVariablesFunctions.hlsl</Transform></span>

```hlsl{none}
// 정점 좌표 변환
VertexPositionInputs vertexInput = GetVertexPositionInputs(input.positionOS.xyz);
float3 positionWS = vertexInput.positionWS; // 월드 좌표
float3 positionVS = vertexInput.positionVS; // 뷰 공간 좌표
float3 positionCS = vertexInput.positionCS; // 클립 공간 좌표
```

Core.hlsl를 포함하면 사용 가능

---

## 텍스처 샘플링

텍스처에서 색을 얻는 방법은 <code class="highlight">tex2D</code> 를 사용하는 방법과 <code class="highlight">Texture2D.Sample</code> 를 사용하는 방법

<div class="grid grid-cols-2 gap-4" style="">
<!--  -->

<!-- tex2Dを使う方法 -->
<div>

<h5>tex2D 사용 (DirectX 셰이더 모델 2 ~ 4 에서 지원)</h5>

```glsl{none}
sampler2D _MainTex;
half4 color = tex2D(_MainTex, uv);
```
</div>
<!-- End tex2Dを使う方法 -->

<!-- Texture2D.Sampleを使う方法 -->
<div>
<h5>Texture2D 사용 (DirectX 셰이더 모델 5 이상에서 지원)</h5>

```hlsl{none}
Texture2D _MainTex;
SamplerState sampler_MainTex;

half4 color = _MainTex.Sample(sampler_MainTex, uv);
```
</div>
<!-- End Texture2D.Sampleを使う方法 -->

</div>

URP에 정의된 매크로는 ShaderModel에 따른 API 차이를 상쇄 해준다.

<!-- Start Grid-->

<div class="grid grid-cols-2 gap-3" style="">

<!-- 1 -->
<div style="">
오래된 API (GLES2 등)

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
신규 API (GLES3 / Vulkan / Metal / D3D11 등)
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

## 텍스처 샘플링 매크로

<div class="grid grid-cols-2 gap-4" style="">

<!--マクロを使わない書き方-->
<div>

<h5>매크로를 이용하지 않는 방법</h5>

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

<h5>매크로를 이용하는 방법</h5>

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

## 셰이더 비교

<!-- <arrow v-click="[1, 2]" x1="400" y1="420" x2="230" y2="330" color="#564" width="3" arrowSize="1" /> -->
<!-- <arrow v-click="[2, 3]" x1="40" y1="420" x2="30" y2="330" color="#564" width="3" arrowSize="1" /> -->

<!-- Grid -->
<div class="grid grid-cols-2 gap-4" style="font-size:92%">

<!-- Build-in RP -->
<div>
<h4>빌트인 RP</h4>

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

## 기타 차이점 : LightMode 태그

URP의 LightMode 태그는 [빌트인RP](https://docs.unity3d.com/ja/2018.4/Manual/SL-PassTags.html)와 다름

<u>빌트인RP</u>에서 사용하는 <code class="highlight">ForwardBase</code> 나 <code class="highlight">ForwardAdd</code> 는
<u>URP</u>에서 <b class="warn">작동하지 않습니다</b>.<br>
대신 <code class="highlight">UniversalForward</code> 나 <code class="highlight">SRPDefaultUnlit</code> 를 사용합니다.

<div class="grid grid-cols-2 gap-4" style="font-size:92%">

<!-- Begin ビルトインRP -->
<div>

<h3>빌트인RP</h3>

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

## LightMode 태그 확장

URP에서 <b>커스텀 LightMode 태그</b>를 추가하고, 이를 <b>C#에서 실행 할 수 있습니다.</b> (ScriptableRenderPass 사용)

<div class="grid grid-cols-2 gap-2"> <!-- Top -->

<div>

패스 태그 <code class="highlight">CustomTag</code> 정의(셰이더)

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

패스 태그<code class="highlight">CustomTag</code>를 지정하여 렌더러 실행 (C#)

```cs{2}
var drawingSettings = RenderingUtils.CreateDrawingSettings(
        new ShaderTagId("CustomTag"), 
        ref renderingData, 
        SortingCriteria.CommonOpaque);
var filteringSettings 
    = new FilteringSettings(RenderQueueRange.all);

// 렌더러 실행(셰이더가 실행됨)
context.DrawRenderers(
    renderingData.cullResults, 
    ref drawingSettings, ref filteringSettings);
```
</div>

</div>

참고: 【URP】 캐릭터의 투명 눈썹 구현 <br>
https://zenn.dev/r_ngtm/articles/urp-transparent-eyebrow

<!--
ScriptableRendererContextのDrawRenderers
を呼ぶと実行される

詳しくは、下記リンクのURLの記事を参照
-->

---
layout: end

---

## 감사합니다.
