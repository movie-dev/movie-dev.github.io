---
layout: post
title: "Part1. 기초 필수 수학 - 벡터대수"
name: movie
link: https://github.com/movie-dev
date: 2026-02-09 23:40:00 +0900
categories: [Direct3D12]
tags: [DirectX12]
mermaid: true
---
# Chapter 1 벡터 대수
## 1.1 벡터
1. 벡터는 크기와 방향을 모두 가진 수량을 가리키는 말이다.
1. 좀 더 공식적으로 벡터값 수량(vector-valued quantity) 이라고 부른다.
1. 예로는 힘(힘은 특정한 방향과 세기로 가해지는데, 세기(strength)가 곧 크기이다.), 변위(한 입자의 최종적인 이동 방향 및 거리), 속도(빠르기와 방향) 등이 있다.
1. 시각적으로 벡터는 방향이 있는 선분, 줄여서 지향 선분(directed line segment) 으로 표시한다. 선분의 길이는 벡터의 크기를 나타내고 선분 끝의 화살표는 벡터의 방향을 뜻한다.
1. 벡터가 그려져 있는 위치는 중요하지 않다. 위치를 바꾸어도 벡터의 크기와 방향은 변하지 않기 때문이다.
1. 따라서 두 벡터는 만일 길이가 같고 같은 방향을 가리키면, 그리고 오직 그럴 때에만 상등이다.

### 1.1.1 벡터와 좌표계
1. 컴퓨터는 벡터들을 기하학적으로 다루지 못하므로, 벡터들을 수치적으로 지정하는 방법이 필요하다.
1. 그 방법은, 공간에 하나의 3차원 좌표계를 도입하고 모든 벡터를 그 꼬리가 그 좌표계의 원점과 일치하도록 이동하는 것이다.
1. 그러면 하나의 벡터를 그 머리의 좌표로 규정할 수 있으며, 프로그램안에서 부동소수점 값 세 개로 표현할 수 있다.
1. 같은 벡터가 있어도 기준계에 따라 좌표가 다르다. 이는 온도에 비유할 수 있다. 물이 끓는 온도는 섭씨에서는 100도이고 화씨에서는 212도이다. 끓는 물의 물리적 온도는 측정 단위의 종류와는 무관하게 일정하다.
1. 이것이 중요한 이유는, 우리가 어떤 베거를 좌표로 규정하거나 식별할 떄 그 좌표가 절대적인 수치들이 아니라 항상 어떤 기준계에 상대적인 수치들임을 뜻하기 때문이다.

### 1.1.2 왼손잡이 좌표계 대 오른손잡이 좌표계
1. Direct3D는 소위 왼손잡이 좌표계(left-handed coordinate system)를 사용한다. 왼손 엄지손가락을 펴서 양의 x축 방향을 가리키게 하고 검지를 양의 y축 방향을 가리키게 하면, 중지의 방향이 대략 양의 z축 방향에 해당한다. 오른손잡이 좌표계는 오른손 엄지손가락을 펴서 양의 x축 방향을 가리키게 하고 검지를 양의 y축 방향을 가리키게 하면, 중지의 방향이 대략 양의 z축 방향에 해당한다.

## 1.2 길이와 단위벡터
1. 기하학적으로 한 벡터의 크기는 해당 지향 선분의 길이이다. 벡터의 크기(길이)는 이중 수직선으로 표기한다. 예를 들어 u의 크기는 u 이다.
1. 벡터 u = (x,y,z)가 주어졌을 때 대수적으로 구해보자. 3차원 벡터의 크기는 피타고라스의 정리를 두 번 적용해서 계산할 수 있다.
![1_Renderer](/assets/img/vector1.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
1. 우선 xz평면에 있는, 직각을 낀 두 변의 길이가 x와 z이고 빗변의 길이가 a인 삼각형을 보자. 피타고라스의 정리에 따르면 $a = \sqrt{x^2 + z^2}$ 이다. 이제 두 변의 길이가 a와 y이고 빗변의 길이가 $\lVert \mathbf{u} \rVert$ 인 삼각형을 보자. 또 다시 피타고라스의 정리를 적용하면, 다음과 같은 벡터의 크기 공식이 나온다.
1. $\lVert \mathbf{u} \rVert = \sqrt{y^2 + a^2} = \sqrt{y^2 + \sqrt{x^2 + z^2}} = \sqrt{x^2 + y^2 + z^2}$
1. 크기가 1인 벡터를 단위벡터(unit vector)라고 부르고, 임의의 벡터를 단위 벡터로 만드는 것을 정규화(normalization)라고 부른다. 벡터의 각 성분을 벡터의 크기로 나누면 벡터가 정규화된다.
1. $\hat{\mathbf{u}}\;=\;\frac{u}{\left\Vert u\right\Vert}=\;\left(\frac{x}{\left\Vert u\right\Vert},\;\frac{y}{\left\Vert u\right\Vert},\;\frac{z}{\left\Vert u\right\Vert}\right)$
1. 이 공식이 맞는지 확인하기 위해, 단위벡터 u 의 길이를 실제로 계산해보자.
1. $\hat{\mathbf{u}}\;=\;\sqrt{\frac{x}{\left\Vert u\right\Vert}^2+\frac{y}{\left\Vert u\right\Vert}^2+\frac{z}{\left\Vert u\right\Vert}^2}=\;\frac{\sqrt{x^2+y^2+z^2}}{\sqrt{\left\Vert u\right\Vert^2}}\;=\;\frac{\left\Vert u\right\Vert}{\left\Vert u\right\Vert}=\;1$

## 1.3 내적
1. 점곱(dot product)이라고도 부르는 내적(inner product)은 스칼라값을 내는 벡터 곱셈의 일종이다. 결과가 스칼라라서 스칼라 곱 이라고 부르기도 한다.
1. $\mathbf{u}=(u_{x},\;u_{y},\;u_{z})$ 이고 $v=(v_{x},\;v_{y},\;v_{z})$ 라고 하자. 그러면 내적은 다음과 같이 정의된다.
1. $\mathbf{u} \cdot \mathbf{v}=(u_{x}v_{x},\;u_{y}v_{y},\;u_{z}v_{z})$
1. 다른 말로 하면 내적은 대응되는 성분들의 곱들의 합이다.
1. 내적의 정의만 봐서는 기하학적 의미가 분명하지 않은데, 코사인 법칙을 적용해 보면 다음과 같은 관계를 찾아낼 수 있다.
1. $\mathbf{u}\cdot\mathbf{v}=\;\left\Vert u\right\Vert\left\Vert v\right\Vert\cos\theta$ `(식 1.4)`
1. 여기서 $\theta$는 벡터 u와 v 사이의, $0\le\theta\le\pi$를 만족하는 각도이다. 따라서 두 벡터의 내적이 두 벡터 사이의 각도의 코사인을 벡터 크기로 비례한 것임을 뜻한다. 특히, u와 v 둘 다 단위벡터일 때 경우 $\mathbf{u}\cdot\mathbf{v}$는 두 벡터 사이의 각도의 코사인이다(즉, $\mathbf{u}\cdot\mathbf{v}=\cos\theta$)
1. 식 1.4로부터 내적의 유용한 기하학적 속성 몇 가지를 이끌어낼 수 있다.
	2. 만일 $\mathbf{u}\cdot\mathbf{v}=0$이면 $\mathbf{u}\perp\mathbf{v}$이다. (즉, 두 벡터는 직교이다.)
	2. 만일 $\mathbf{u}\cdot\mathbf{v}>0$이면 두 벡터 사이의 각도 $\theta$는 90도보다 작다(즉, 두 벡터는 예각을 이룬다).
	2. 만일 $\mathbf{u}\cdot\mathbf{v}<0$이면 두 벡터 사이의 각도 $\theta$는 90도보다 크다(즉, 두 벡터는 예각을 이룬다).
1. 예 1.4
	2. $\mathbf{u}=(1,\;2,\;3)$이고 $\mathbf{v}=(-4,0,-1)$ 이라고 할 때, $\mathbf{u}$와 $\mathbf{v}$ 사이의 각도를 구해 보자.
	2. $\mathbf{u}\cdot\mathbf{v}=\left(1,\;2,\;3\right)\cdot\;\left(-4,\;0,-1\right)=-4-3=-7$
	2. $\left\Vert u\right\Vert=\sqrt{1^2+2^2+3^2}=\sqrt{14}$
	2. $\left\Vert v\right\Vert=\sqrt{\left(-4\right)^2+0^2+\left(-1\right)^2}=\sqrt{17}$
	2. $\cos\theta=\frac{u\cdot v}{\left\Vert u\right\Vert\left\Vert v\right\Vert}=\frac{-7}{\sqrt{14\sqrt{17}}}$
	2. $\theta=\cos^{-1}\frac{-7}{\sqrt{14\sqrt{17}}}\approx117^{\circ}$
1. 예 1.5 ($\mathbf{n}$에 대한 $\mathbf{v}$의 직교투영)
	2. ![1_Renderer](/assets/img/vector2.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
	2. 벡터 $\mathbf{v}$와 단위벡터 $\mathbf{n}$이 주어졌을 때 $\mathbf{p}$를 내적을 이용해서 $\mathbf{v}$와 $\mathbf{n}$으로 표현하는 공식을 구해 보자
	2. 그림을 보면 p = kn을 만족하는 스칼라 k가 존재함을 알 수 있다. 더 나아가서, $\left\Vert n\right\Vert=1$ 이므로 반드시 $\left\Vert p\right\Vert=\left\Vert kn\right\Vert=\left\vert k\right\vert\left\Vert n\right\Vert=\left\vert k\right\vert$이다. (k는 오직 p와 n이 반대 방향일 때에만 음수임을 주목할 것.) 삼각함수 법칙들을 적용하면 $k=\left\Vert v\right\Vert\cos\theta$가 나온다.
	2. 따라서, $p=kn=\left\Vert v\right\Vert\cos\theta$이다. 그런데 n은 단위벡터이므로, 이를 다음과 같이 표현할 수도 있다.
	2. $p=kn=\left(\Vert v\right\Vert\cos\theta)n=\left(\left\Vert v\right\Vert\cdot1\cos\theta\right)n=\left(\left\Vert v\left\Vert n\right\Vert\right\Vert\cos\theta\right)n=\left(v\cdot n\right)n$
	2. 특히 이 공식에 따르면 $k=v\cdot n$ 이다. 이는 n이 단위벡터일 때 $v\cdot n$의 기하학적 의미를 말해준다. 이러한 p를 n에 대한 v의 직교투영(orthographic projection; 또는 정사영) 이라고 부르며, 흔히 다음과 같이 표기한다.
	2. $p=proj_{n}\left(v\right)$
	2. v를 하나의 힘으로 간주한다면 p는 힘 v 중에서 방향 n으로 작용하는 부분이라고 할 수 있다.
	2. 이와 비슷하게, 벡터 $w=perp_{n}\left(v\right)=v-p$는 힘 v 중에서 n의 수직 방향으로 작용하는 부분이다.
	2. v = p + w임을 주목하기 바란다. 즉, v는 두 직교벡터 p와 w의 합으로 분해된다. n이 단위 길이가 아니면, 먼저 n을 정규화해서 단위 길이로 만들면 된다. 위의 투영 공식에서 n을 단위 벡터 $\frac{n}{\left\Vert n\right\Vert}$으로 대체하면 다음과 같은 좀 더 일반적인 투영 공식이 나온다.
	2. $p=proj_{n}\left(v\right)=\left(v\cdot\frac{n}{\left\Vert n\right\Vert}\right)\frac{n}{\left\Vert n\right\Vert}=\frac{v\cdot n}{\left\Vert n\right\Vert^2}n$

### 1.3.1 직교화
1. 벡터 집합 $\left\lbrace v_0,\;\ldots\;v_{n-1}\right\rbrace$의 모든 벡터가 단위 길이이고 서로 직교일 때(즉, 집합의 모든 벡터가 다른 모든 벡터와 수직일 때), 그러한 벡터 집합을 정규직교(orthonormal) 집합이라고 부른다. 정규직교에 가깝지만 완전히 정규직교는 아닌 경우도 흔히 만나게 된다. 그런 벡터 집합을 정규직교벡터 집합으로 만드는 것을 직교화(orthogonalization) 라고 부른다.
1. ![1_Renderer](/assets/img/vector3.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
1. 2차원 경우 벡터 집합 $\left\lbrace v_0,\;v_1\right\rbrace$을 직교화해서 정규직교 집합 $\left\lbrace w_0,\;w_1\right\rbrace$을 얻는 과정이 [그림 1.11]에 나와 있다. 우선 $w_0=v_0$으로 시작해서, 벡터 $v_1$이 $w_0$과 직교가 되게 만든다.
1. 이를 위해, $w_0$의 방향으로 작용하는 부분을 $v_1$에서 뺀다.
1. $w_1=v_1-proj_{w_0}\left(v1\right)$
1. 이제 서로 직교인 벡터들의 집합 $\left\lbrace w_0,w_1\right\rbrace$이 만들어졌다. 마지막으로 $w_0$과 $w_1$을 정규화해서 단위 길이로 만들면 정규직교 집합이 완성된다.
1. 3차원의 경우도 마찬가지로 같은 원칙을 적용하면 된다.
1. $w_1=v_1-proj_{w_0}\left(v1\right)$
1. $w_2=v_2-proj_{w_0}\left(v2\right)-proj_{w_1}\left(v2\right)$
1. 이를 일반화해서, n개의 벡터들의 집합 $\left\lbrace v_0,\;\ldots\;v_{n-1}\right\rbrace$을 정규직교 집합 $\left\lbrace w_0,\;\ldots\;w_{n-1}\right\rbrace$으로 직교화할 때에는 그람-슈미트 직교화(Gram-Schmidt Orthogonalization)라고 하는 공정을 적용한다. 그람-슈미트 직교화 공정은 다음과 같다.
1. 기본 단계: $w_0=v_0$으로 설정한다.
1. $1\le i\le n-1$에 대해 $w_{i}=v_{i}-\sum_{j=0}^{i-1}proj_{w_{j}}\left(v_{i}\right)$로 설정한다.
1. 정규화 단계: $w_{i}=\frac{w_{i}}{\left\Vert w_{i}\right\Vert}$로 설정한다.

## 1.4 외적
1. 또 다른 벡터 곱셈으로 가위곱(cross product) 또는 외적(outer product)이라는 것이 있다. 결과가 스칼라인 내적과는 달리 외적의 결과는 벡터이다. 또한, 외적은 오직 3차원 벡터에 대해서만 정의된다. 두 3차원 벡터 u와 v의 외적을 취하면 u와 v 모두에 직교인 또 다른 벡터 w가 나온다.
1. $u=\left(u_{x},\;u_{y},\;u_{z}\right)$이고 $v=\left(v_{x},\;v_{y},\;v_{z}\right)$ 라고 할 때 둘의 외적은 다음과 같이 정의된다.
1. $w=u\times v=\left(u_{y}v_{z}-u_{z}v_{y},\;u_{z}v_{x}-u_{x}v_{z},\;u_{x}v_{y}-u_{y}v_{x}\right)$ `식 1.5`
1. 예 1.6
	2. $u=\left(2,1,3\right)$이고 $v=\left(2,0,0\right)$이라고 할 때 $w=u\times v$와 $z=v\times u$를 계산하고 w가 u와 v에 직교임을 확인해 보자.
	2. $w=u\times v$
		3. $\left(2,1,3\right)\times\left(2,0,0\right)$
		3. $\left(1\cdot0-3\cdot0,\;3\cdot2-2\cdot0,\;2\cdot0-1\cdot2\right)$
		3. $\left(0,-6,2\right)$
	2. $z=v\times u$
		3. $\left(2,0,0\right)\times\left(2,1,3\right)$
		3. $\left(0\cdot3-0\cdot1,\;0\cdot2-2\cdot3,\;2\cdot1-0\cdot2\right)$
		3. $\left(0,6,-2\right)$
	2. 외적에는 교환법칙이 성립하지 않는다. 실제로 $u\times v$ = $-v\times u$ 임을 증명하는 것이 가능하다.
	2. w가 u에 직교인지, 그리고 v에 직교인지를 확인하는 방법은 $u\cdot v=0$이면 $u\perp v$이다. 실제로 계산해보면
	2. $w\cdot u=\left(0,6,-2\right)\cdot\left(2,1,3\right)=\left(0\cdot2+6\cdot1+\left(-2\right)\cdot3\right)=0$
	2. $w\cdot v=\left(0,6,-2\right)\cdot\left(2,0,0\right)=\left(0\cdot2+6\cdot0+\left(-2\right)\cdot0\right)=0$
	
### 1.4.1 외적을 이용한 직교화
1. ![1_Renderer](/assets/img/vector4.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }
1. $w_0=\frac{v_0}{\left\Vert v_0\right\Vert}$으로 설정한다.
1. $w_2=\frac{w_0\times w_1}{\left\Vert w_0\times w_1\right\Vert}$로 설정한다.
1. $w_1=w_2\times w_0$ 으로 설정한다. $w_2\perp w_0$이고 $\left\Vert w_2\right\Vert=\left\Vert w_0\right\Vert=1$이므로 $\left\Vert w_2\times w_0\right\Vert=1$이다. 따라서 이 마지막 단계에서는 더 이상의 정규화가 필요하지 않다.

## 1.5 점
1. 벡터를 이용해 3차원 공간 안의 한 위치를 나타내는 데 사용할 수 있다. 그러한 벡터를 위치벡터라고 부른다.
1. 점을 벡터로 표현하는 방식의 한가지 부작용은, 점에 대해서는 의미가 없는 벡터 연산을 점에 적용하는 실수를 저지를 여지가 생긴다는 것이다. 예를 들어 기하학적으로 두 점의 합은 말이 되지 않는다. 그러나 점에 대해서도 의미 있게 적용할 수 있는 벡터 연산들도 존재한다. 예를 들어 두 점의 차 q - p를, p에서 q로 가는 벡터라고 정의할 수 있다. 또한, 점 p 더하기 벡터 v를 p의 위치를 v만큼 옮겼을 때(‘변위’) 도달하는 점 q라고 정의하는 것도 가능하다.

## 1.6 DirectXMath 라이브러리의 벡터
1. Direct3D 응용 프로그램을 위한 표준적인 3차원 수학 라이브러리는 DirectXMath이다. 이 라이브러리는 SSE2(Streaming SIMD Extensions 2) 명령 집합을 활용한다. SIMD 명령들은 128비트 너비의 SIMD(single instruction multiple data) 레지스터들을 이용해서 32비트 float 또는 int 네 개를 단번에 처리할 수 있다.

### 1.6.1 벡터 형식들
1. DirectXMath에서 핵심 벡터 형식은 SIMD 하드웨어 레지스터에 대응되는 XMVECTOR이다. x64플랫폼에서, 그리고 SSE2가 활성화된 x86 플랫폼에서 이 형식은 다음과 같이 정의된다.
1. typedef __m128 XMVECTOR;
1. XMVECTOR는 16바이트 경계에 정합(alignment) 되어야 하는데, 지역 변수와 전역 변수에서는 그러한 정합이 자동으로 일어난다. 클래스 자료 멤버에는 이 형식 대신 XMFLOAT2나 XMFLOAT3, XMFLOAT4를 사용하는 것이 권장된다.
1. 그러나 이 형식들을 계산에 직접 사용하면 SIMD의 장점을 취할 수 없다. SIMD를 활용하려면 이 형식들의 인스턴스를 XMVECTOR 형식으로 변환해야 한다.
1. 다행히 라이브러리에서 모두 지원한다.

### 1.6.2 매개변수 전달
1. 효율성을 위해서는 XMVECTOR 값이 스택이 아니라 SSE/SSE2 레지스터를 통해서 함수에 전달되게 해야 한다. 그런 식으로 전달할 수 있는 인수의 개수는 플랫폼과 컴파일러에 따라 다르다.
1. 플랫폼/컴파일러에 대한 의존성을 없애기 위해서는 XMVECTOR 매개변수에 대해 FXMVECTOR, GXMVECTOR, HXMVECTOR, CXMVECTOR라는 형식들을 사용해야 한다.
1. 또한 SSE/SSE2 레지스터 활용을 위한 호출 규약 역시 컴파일러에 따라 다를 수 있는데, 그러한 의존성을 없애려면 함수 이름 앞에 반드시 XM_CALLCONV라는 호출 규약 지시자를 붙여야한다.
	2. 처음 세 XMVECTOR 매개변수에는 반드시 FXMVECTOR 형식을 지정해야 한다.
	2. 넷째 XMVECTOR 매개변수에는 GXMVECTOR 형식을 지정해야 한다.
	2. 다섯째와 여섯째 XMVECTOR 매개변수에는 반드시 HXMVECTOR 형식을 지정해야 한다.
	2. 그 이상의 XMVECTOR 매개변수들에는 반드시 CXMVECTOR 형식을 지정해야 한다.
1. 그런데 생성자에 대해서는 조금 다른 규칙이 적용됨을 주의하기 바란다. XMVECTOR 형식의 인수들을 받는 생성자를 작성할 떄 처음 세 XMVECTOR 매개변수에는 FXMVECTOR를, 그 나머지에는 CXMVECTOR를 사용하라고 권한다. 또한 생성자는 XM_CALLCONV 호출 규약 지시자를 사용하지 말아야 한다.
1. 지금까지 말한 XMVECTOR 매개변수 전달 규칙은 ‘입력’ 매개변수들에 적용된다. ‘출력’ XMVECTOR 매개변수는 SSE/SSE2 레지스터를 사용하지 않으므로, 그냥 XMVECTOR가 아닌 매개변수들과 동일하게 취급된다.

### 1.6.3 상수 벡터
1. 상수 XMVECTOR 인스턴스에는 반드시 XMVECTORF32 형식을 사용해야 한다. XMVECTORF32는 16바이트 경계에 정합되는 구조체로, XMVECTOR로의 변환 연산자들을 제공한다.
1. 정수 자료를 담은 상수 XMVECTOR를 생성하고 싶으면 XMVECTORU32를 사용하면 된다.

