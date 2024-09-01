---
layout: post
title: "#1 DirectX 11 기본 게임 프레임워크"
name: movie
link: https://github.com/movie-dev
date: 2024-08-26 22:10:00 +0900
categories: [Renderer]
tags: [DirectX11]
mermaid: true
---
## 렌더러 개발 시작
엔진 관련 내용에 대해 자세히 공부 해보고 싶어서 DirectX 11을 이용하여 렌더러를 만들면서 관련 지식을 쌓아보려고 합니다. 모든 코드는 [Mao Renderer](https://github.com/movie-dev/MaoRenderer){:target="_blank"} public 저장소에 기록하려고 합니다. 기억이 가물가물해서 인터넷 검색과 서적을 보며 첫번째 코드를 커밋 했습니다.

## 게임 프레임워크 코드 추가
MaoGameFramework 전역 클래스를 추가 했습니다. 이 클래스에서는 하드웨어와 통신할 수 있도록 디바이스 인터페이스들을 관리하며 게임 객체의 업데이트와 렌더링을 담당 시키려고 합니다.
```c++
	class MaoGameFramework
	{
	public:
		bool OnCreate(HINSTANCE hInstance, HWND hMainWnd);
		void OnDestroy();

		bool CreateDirect3DDisplay();
		bool CreateRenderTargetDepthStencilView();

		void FrameAdvance();

	private:
		HINSTANCE Instance{ nullptr };
		HWND Wnd{ nullptr };

		ID3D11Device* D3DDevice{ nullptr };
		ID3D11DeviceContext* D3DDeviceContext{ nullptr };
		IDXGISwapChain* DXGISwapChain{ nullptr };
		ID3D11RenderTargetView* D3DRenderTargetView{ nullptr };
		ID3D11Texture2D* D3DDepthStencilBuffer{ nullptr };
		ID3D11DepthStencilView* D3DDepthStencilView{ nullptr };

		int WndClientWidth{ 0 };
		int WndClientHeight{ 0 };
	};
```

DirectX 렌더링에 필요한 `ID3D11Device`, `ID3D11DeviceContext`, `IDXGISwapChain`, `ID3D11RenderTargetView`, `ID3D11DepthStencilView` 등을 생성합니다. 이 인터페이스들에 대해 정확히 알지는 못합니다. 대충 어떤 역할을 하는지만 어렴풋이 알고 있는데 계속 개발하면서 내용을 추가해야 될 것 같습니다. 

### D3DDevice / D3DDeviceContext / DXGISwapChain 생성
인터페이스 생성에 필요한 구조체 정보도 굉장히 복잡합니다. 변수 이름으로 봐서는 렌더링 버퍼의 크기와 포맷, FPS 등을 지정하는 것 같습니다.
```c++
	bool MaoGameFramework::CreateDirect3DDisplay()
	{
		RECT rcClient;
		::GetClientRect(Wnd, &rcClient);
		WndClientWidth = rcClient.right - rcClient.left;
		WndClientHeight = rcClient.bottom - rcClient.top;

		DXGI_SWAP_CHAIN_DESC dxgiSwapChainDesc;
		::ZeroMemory(&dxgiSwapChainDesc, sizeof(dxgiSwapChainDesc));
		dxgiSwapChainDesc.BufferCount = 1;
		dxgiSwapChainDesc.BufferDesc.Width = WndClientWidth;
		dxgiSwapChainDesc.BufferDesc.Height = WndClientHeight;
		dxgiSwapChainDesc.BufferDesc.Format = DXGI_FORMAT_R8G8B8A8_UNORM;
		dxgiSwapChainDesc.BufferDesc.RefreshRate.Numerator = 60;
		dxgiSwapChainDesc.BufferDesc.RefreshRate.Denominator = 1;
		dxgiSwapChainDesc.BufferUsage = DXGI_USAGE_RENDER_TARGET_OUTPUT;
		dxgiSwapChainDesc.OutputWindow = Wnd;
		dxgiSwapChainDesc.SampleDesc.Count = 1;
		dxgiSwapChainDesc.SampleDesc.Quality = 0;
		dxgiSwapChainDesc.Windowed = TRUE;

		UINT dwCreateDeviceFlags = 0;
	#ifdef _DEBUG
		dwCreateDeviceFlags |= D3D11_CREATE_DEVICE_DEBUG;
	#endif

		constexpr D3D_DRIVER_TYPE d3dDriverTypes[] =
		{
			D3D_DRIVER_TYPE_HARDWARE,
			D3D_DRIVER_TYPE_WARP,
			D3D_DRIVER_TYPE_REFERENCE
		};
		constexpr auto DriverTypes = sizeof(d3dDriverTypes) / sizeof(D3D_DRIVER_TYPE);

		constexpr D3D_FEATURE_LEVEL d3dFeatureLevels[] =
		{
			D3D_FEATURE_LEVEL_11_0,
			D3D_FEATURE_LEVEL_10_1,
			D3D_FEATURE_LEVEL_10_0,
		};
		constexpr auto FeatureLevels = sizeof(d3dFeatureLevels) / sizeof(D3D_FEATURE_LEVEL);

		HRESULT Result = S_OK;
		D3D_DRIVER_TYPE d3dDriverType = D3D_DRIVER_TYPE_NULL;
		D3D_FEATURE_LEVEL d3dFeatureLevel = D3D_FEATURE_LEVEL_11_0;
		for (UINT i = 0; i < DriverTypes; i++)
		{
			d3dDriverType = d3dDriverTypes[i];
			if (SUCCEEDED(Result = D3D11CreateDeviceAndSwapChain(nullptr, d3dDriverType, NULL, dwCreateDeviceFlags, d3dFeatureLevels, FeatureLevels, D3D11_SDK_VERSION, &dxgiSwapChainDesc, &DXGISwapChain, &D3DDevice, &d3dFeatureLevel, &D3DDeviceContext)))
			{
				break;
			}
		}

		if (!CreateRenderTargetDepthStencilView())
		{
			return false;
		}

		return true;
	}
```

### DepthStencilView / DepthStencilBuffer 생성
장면의 깊이나 마스킹 효과등에 사용하는 것으로 어렴풋이 기억하고 있습니다. 이 부분도 꽤 어려운 주제이므로 더 공부해서 따로 페이지를 만들어야 할 것 같습니다.
```c++
bool MaoGameFramework::CreateRenderTargetDepthStencilView()
{
	HRESULT hResult = S_OK;

	ID3D11Texture2D* d3dBackBuffer;
	if (FAILED(hResult = DXGISwapChain->GetBuffer(0, __uuidof(ID3D11Texture2D), reinterpret_cast<LPVOID *>(&d3dBackBuffer))))
	{
		return false;
	}
	if (FAILED(hResult = D3DDevice->CreateRenderTargetView(d3dBackBuffer, NULL, &D3DRenderTargetView)))
	{
		return false;
	}
	if (d3dBackBuffer)
	{
		d3dBackBuffer->Release();
	}
	
	// Create depth stencil texture
	D3D11_TEXTURE2D_DESC d3dDepthStencilBufferDesc;
	ZeroMemory(&d3dDepthStencilBufferDesc, sizeof(D3D11_TEXTURE2D_DESC));
	d3dDepthStencilBufferDesc.Width = WndClientWidth;
	d3dDepthStencilBufferDesc.Height = WndClientHeight;
	d3dDepthStencilBufferDesc.MipLevels = 1;
	d3dDepthStencilBufferDesc.ArraySize = 1;
	d3dDepthStencilBufferDesc.Format = DXGI_FORMAT_D24_UNORM_S8_UINT;
	d3dDepthStencilBufferDesc.SampleDesc.Count = 1;
	d3dDepthStencilBufferDesc.SampleDesc.Quality = 0;
	d3dDepthStencilBufferDesc.Usage = D3D11_USAGE_DEFAULT;
	d3dDepthStencilBufferDesc.BindFlags = D3D11_BIND_DEPTH_STENCIL;
	d3dDepthStencilBufferDesc.CPUAccessFlags = 0;
	d3dDepthStencilBufferDesc.MiscFlags = 0;
	if (FAILED(hResult = D3DDevice->CreateTexture2D(&d3dDepthStencilBufferDesc, NULL, &D3DDepthStencilBuffer)))
	{
		return false;
	}

	// Create the depth stencil view
	D3D11_DEPTH_STENCIL_VIEW_DESC d3dDepthStencilViewDesc;
	ZeroMemory(&d3dDepthStencilViewDesc, sizeof(D3D11_DEPTH_STENCIL_VIEW_DESC));
	d3dDepthStencilViewDesc.Format = d3dDepthStencilBufferDesc.Format;
	d3dDepthStencilViewDesc.ViewDimension = D3D11_DSV_DIMENSION_TEXTURE2D;
	d3dDepthStencilViewDesc.Texture2D.MipSlice = 0;
	if (FAILED(hResult = D3DDevice->CreateDepthStencilView(D3DDepthStencilBuffer, &d3dDepthStencilViewDesc, &D3DDepthStencilView)))
	{
		return false;
	}

	D3DDeviceContext->OMSetRenderTargets(1, &D3DRenderTargetView, D3DDepthStencilView);

	return true;
}
```

## 화면 결과
D3DRenderTargetView 와 D3DDepthStencilView 를 Clear하고 DXGISwapChain 을 Present한 결과입니다.
```c++
	void MaoGameFramework::FrameAdvance()
	{
		constexpr float fClearColor[4] = { 0.0f, 0.125f, 0.3f, 1.0f }; 
		if (D3DRenderTargetView)
		{
			D3DDeviceContext->ClearRenderTargetView(D3DRenderTargetView, fClearColor);
		}
		if (D3DDepthStencilView)
		{
			D3DDeviceContext->ClearDepthStencilView(D3DDepthStencilView, D3D11_CLEAR_DEPTH|D3D11_CLEAR_STENCIL, 1.0f, 0);
		}

		DXGISwapChain->Present(0, 0);
	}
```

![1_Renderer](/assets/img/1_renderer.png){:style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;" }