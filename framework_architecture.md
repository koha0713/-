# DirectX Game Framework - アーキテクチャドキュメント

## 1. フレームワーク概要

本フレームワークは、DirectX11を用いた3Dゲーム開発のための基盤システムです。
コンポーネントベースアーキテクチャ、シーン管理、物理演算、リソース管理を統合しています。

---

## 2. 全体アーキテクチャ図

```mermaid
graph TB
    subgraph "Application Layer"
        Main[main.cpp]
        App[Application]
        Game[Game]
    end
    
    subgraph "Manager Layer"
        SceneMgr[SceneManager]
        ResMgr[ResourceManager]
        PhysMgr[PhysicsManager]
        SoundMgr[SoundManager]
        IOMgr[IOManager]
        DataMgr[DataManager]
    end
    
    subgraph "Scene Layer"
        SceneBase[SceneBase]
        SceneTitle[SceneTitle]
        SceneGame[SceneGame]
        SceneResult[SceneResult]
    end
    
    subgraph "GameObject Layer"
        GameObject[GameObject]
        Transform[Transform]
        Component[Component]
    end
    
    subgraph "Component Systems"
        Render[Rendering Components]
        Physics[Physics Components]
        Custom[Custom Components]
    end
    
    subgraph "Graphics Layer"
        Renderer[Renderer]
        Shader[Shader]
        Texture[Texture]
        Mesh[StaticMesh]
        Material[Material]
    end
    
    subgraph "Utility Layer"
        Timer[Timer]
        Singleton[Singleton]
        Common[SystemCommon]
    end
    
    Main --> App
    App --> Game
    Game --> SceneMgr
    Game --> ResMgr
    Game --> PhysMgr
    Game --> SoundMgr
    Game --> IOMgr
    Game --> DataMgr
    Game --> Timer
    
    SceneMgr --> SceneBase
    SceneBase --> SceneTitle
    SceneBase --> SceneGame
    SceneBase --> SceneResult
    
    SceneBase --> GameObject
    GameObject --> Transform
    GameObject --> Component
    
    Component --> Render
    Component --> Physics
    Component --> Custom
    
    Render --> Renderer
    Render --> Shader
    Render --> Texture
    Render --> Mesh
    Render --> Material
    
    ResMgr --> Shader
    ResMgr --> Texture
    ResMgr --> Mesh
    
    PhysMgr --> Physics
```

---

## 3. 主要クラス図

### 3.1 コアシステム

```mermaid
classDiagram
    class Application {
        -HWND m_hWnd
        -uint32_t m_Width
        -uint32_t m_Height
        +Application(width, height)
        +Run()
        +GetWidth() uint32_t
        +GetHeight() uint32_t
        +GetWindow() HWND
        -InitApp() bool
        -UninitApp()
        -MainLoop()
    }
    
    class Game {
        -Camera m_Camera
        -Timer m_Timer
        +Game()
        +Init()
        +Update()
        +Draw()
        +Uninit()
        +GetDeltaTime() float$
        +GetTotalTime() float$
        +GetFPS() float$
    }
    
    class Timer {
        -TimePoint m_LastTime
        -float m_DeltaTime
        -float m_TotalTime
        -unsigned int m_FrameCount
        -float m_CurrentFps
        +Timer()
        +Update()
        +GetDeltaTime() float
        +GetTotalTime() float
        +GetFPS() float
        +Reset()
    }
    
    Application --> Game : creates
    Game --> Timer : uses
```

### 3.2 マネージャー層

```mermaid
classDiagram
    class SceneManager {
        -SCENE m_currentScene
        -unique_ptr~Scene~ m_scene
        +Init()
        +UnInit()
        +Update()
        +Draw()
        +Draw(Camera*)
        +ChangeScene(SCENE)
    }
    
    class ResourceManager {
        -map~string, shared_ptr~Texture~~ m_TextureCache
        -map~string, shared_ptr~StaticMesh~~ m_MeshCache
        -map~string, shared_ptr~Shader~~ m_ShaderCache
        +LoadTexture(filepath) shared_ptr~Texture~
        +LoadMesh(filepath, textureDir) shared_ptr~StaticMesh~
        +LoadShader(vsPath, psPath) shared_ptr~Shader~
        +ClearAll()
    }
    
    class PhysicsManager {
        -vector~Collider*~ m_Colliders
        -unordered_set~CollisionPair~ m_PreviousCollisions
        -bool m_LayerCollisionMatrix[32][32]
        -Vector3 m_Gravity
        +Init()
        +Update()
        +RegisterCollider(Collider*)
        +UnregisterCollider(Collider*)
        +SetLayerCollision(layer1, layer2, enable)
        +GetLayerCollision(layer1, layer2) bool
    }
    
    class SoundManager {
        -float m_volumeBGM
        -float m_volumeSE
        -SOUND_LABEL m_currentBGM
        +Init()
        +PlayBGM(SOUND_LABEL)
        +PlaySE(SOUND_LABEL)
        +Stop(SOUND_LABEL)
        +SetVolumeBGM(float)
        +SetVolumeSE(float)
    }
    
    class IOManager {
        -INPUT_MODE m_mode
        -Input m_Input
        -map m_keyBindTable
        +Init()
        +Update()
        +GetKeyDown(INPUT_TYPE) bool
        +GetKeyPress(INPUT_TYPE) bool
        +GetKeyUp(INPUT_TYPE) bool
    }
```

### 3.3 GameObjectシステム

```mermaid
classDiagram
    class GameObject {
        -Transform m_Transform
        -vector~unique_ptr~Component~~ m_Components
        -unordered_map m_ComponentMap
        -bool m_Active
        -string m_Tag
        +AddComponent~T~(...) T*
        +GetComponent~T~() T*
        +RemoveComponent~T~()
        +Update()
        +Draw(Camera*)
        +DrawLayer(Camera*, RenderLayer)
        +Uninit()
        +SetActive(bool)
        +IsActive() bool
    }
    
    class Transform {
        -Vector3 m_Position
        -Vector3 m_Rotation
        -Vector3 m_Scale
        +SetPosition(Vector3)
        +SetRotation(Vector3)
        +SetScale(Vector3)
        +GetPosition() Vector3
        +GetRotation() Vector3
        +GetScale() Vector3
        +GetWorldMatrix() Matrix
        +Translate(Vector3)
        +Rotate(Vector3)
        +GetForward() Vector3
        +GetUp() Vector3
        +GetRight() Vector3
    }
    
    class Component {
        #GameObject* m_pOwner
        #bool m_Enabled
        #RenderLayer m_RenderLayer
        +Init()*
        +Update()*
        +Draw(Camera*)*
        +Uninit()*
        +SetOwner(GameObject*)
        +GetOwner() GameObject*
        +SetEnabled(bool)
        +IsEnabled() bool
    }
    
    GameObject *-- Transform
    GameObject "1" *-- "*" Component
    Component --> GameObject : owns
```

### 3.4 コンポーネントシステム

```mermaid
classDiagram
    class Component {
        <<abstract>>
        +Init()*
        +Update()*
        +Draw(Camera*)*
    }
    
    class MeshRendererComponent {
        -shared_ptr~StaticMesh~ m_Mesh
        -shared_ptr~Shader~ m_Shader
        -vector~unique_ptr~Material~~ m_Materials
        -MeshRenderer m_Renderer
        +SetMesh(shared_ptr~StaticMesh~)
        +SetShader(shared_ptr~Shader~)
        +Draw(Camera*)
    }
    
    class Collider {
        <<abstract>>
        #ColliderType m_Type
        #Vector3 m_Center
        #bool m_IsTrigger
        #int m_Layer
        +CheckCollision(Collider*, CollisionInfo&) bool*
        +OnCollisionEnter(CollisionInfo)
        +OnCollisionStay(CollisionInfo)
        +OnCollisionExit(CollisionInfo)
    }
    
    class SphereCollider {
        -float m_Radius
        +CheckCollision(Collider*, CollisionInfo&) bool
        +SetRadius(float)
    }
    
    class AABBCollider {
        -Vector3 m_Size
        +CheckCollision(Collider*, CollisionInfo&) bool
        +GetMin() Vector3
        +GetMax() Vector3
    }
    
    class Rigidbody {
        -Vector3 m_Velocity
        -Vector3 m_Acceleration
        -float m_Mass
        -float m_Drag
        -bool m_UseGravity
        +AddForce(Vector3)
        +AddImpulse(Vector3)
        +SetVelocity(Vector3)
    }
    
    Component <|-- MeshRendererComponent
    Component <|-- Collider
    Component <|-- Rigidbody
    Collider <|-- SphereCollider
    Collider <|-- AABBCollider
```

### 3.5 グラフィックスシステム

```mermaid
classDiagram
    class Renderer {
        <<static>>
        -ID3D11Device* m_pDevice
        -ID3D11DeviceContext* m_pDeviceContext
        -IDXGISwapChain* m_pSwapChain
        -ID3D11RenderTargetView* m_pRenderTargetView
        -ID3D11DepthStencilView* m_pDepthStencilView
        +Init() HRESULT$
        +DrawStart()$
        +DrawEnd()$
        +SetWorldMatrix(Matrix*)$
        +SetViewMatrix(Matrix*)$
        +SetProjectionMatrix(Matrix*)$
        +CreateVertexShader(...)$
        +CreatePixelShader(...)$
    }
    
    class Shader {
        -ComPtr~ID3D11VertexShader~ m_pVertexShader
        -ComPtr~ID3D11PixelShader~ m_pPixelShader
        -ComPtr~ID3D11InputLayout~ m_pVertexLayout
        +Create(vsPath, psPath)
        +SetGPU()
    }
    
    class Texture {
        -string m_texname
        -ComPtr~ID3D11ShaderResourceView~ m_srv
        -int m_width
        -int m_height
        +Load(filename) bool
        +LoadFromMemory(data, len) bool
        +SetGPU()
    }
    
    class StaticMesh {
        -vector~VERTEX_3D~ m_vertices
        -vector~unsigned int~ m_indices
        -vector~MATERIAL~ m_materials
        -vector~SUBSET~ m_subsets
        -vector~unique_ptr~Texture~~ m_textures
        +Load(filename, textureDirectory)
        +GetVertices() vector~VERTEX_3D~
        +GetIndices() vector~unsigned int~
        +GetMaterials() vector~MATERIAL~
        +GetSubsets() vector~SUBSET~
    }
    
    class Material {
        -MATERIAL m_Material
        -ComPtr~ID3D11Buffer~ m_pConstantBufferMaterial
        +Create(MATERIAL) bool
        +SetGPU()
        +SetDiffuse(Vector4)
        +SetAmbient(Vector4)
    }
    
    Renderer ..> Shader
    Renderer ..> Texture
    StaticMesh *-- Material
    StaticMesh *-- Texture
```

### 3.6 シーンシステム

```mermaid
classDiagram
    class SceneBase {
        <<abstract>>
        #bool m_isInitialized
        #bool m_isActive
        #vector~unique_ptr~GameObject~~ m_GameObjects
        +Init()*
        +UnInit()*
        +Update()*
        +Draw()*
        +Draw(Camera*)*
        #AddGameObject~T~() T*
        #UpdateObjectList()
        #DrawObjectList(Camera*)
        #DrawLayer(Camera*, RenderLayer)
        #FindGameObjectWithTag(string) GameObject*
    }
    
    class SceneTitle {
        +Init()
        +UnInit()
        +Update()
        +Draw()
        +Draw(Camera*)
    }
    
    class SceneGame {
        -float m_GameTime
        +Init()
        +UnInit()
        +Update()
        +Draw()
        +Draw(Camera*)
    }
    
    class SceneResult {
        -float m_DisplayTime
        -Camera* m_pCamera
        +Init()
        +UnInit()
        +Update()
        +Draw()
        +Draw(Camera*)
    }
    
    SceneBase <|-- SceneTitle
    SceneBase <|-- SceneGame
    SceneBase <|-- SceneResult
    SceneBase "1" *-- "*" GameObject
```

---

## 4. システムフロー図

### 4.1 アプリケーション初期化フロー

```mermaid
sequenceDiagram
    participant Main
    participant Application
    participant Game
    participant Managers
    participant Renderer
    
    Main->>Application: new Application(width, height)
    Main->>Application: Run()
    Application->>Application: InitApp()
    Application->>Renderer: Init()
    Application->>Game: Init()
    Game->>Managers: PHYSICS_MANAGER.Init()
    Game->>Managers: SCENE_MANAGER.Init()
    Game->>Managers: RESOURCE_MANAGER.Init()
    Game->>Managers: SOUND_MANAGER.Init()
    Game->>Managers: IO_MANAGER.Init()
    Game->>Game: m_Camera.Init()
    Application->>Application: MainLoop()
```

### 4.2 メインループフロー

```mermaid
sequenceDiagram
    participant Application
    participant Game
    participant Managers
    participant Scene
    participant GameObjects
    
    loop Every Frame
        Application->>Application: PeekMessage()
        Application->>Game: Update()
        Game->>Game: m_Timer.Update()
        Game->>Managers: IO_MANAGER.Update()
        Game->>Managers: PHYSICS_MANAGER.Update()
        Game->>Managers: SCENE_MANAGER.Update()
        Managers->>Scene: currentScene->Update()
        Scene->>GameObjects: UpdateObjectList()
        GameObjects->>GameObjects: Each GameObject->Update()
        Game->>Game: m_Camera.Update()
        
        Application->>Game: Draw()
        Game->>Renderer: DrawStart()
        Game->>Managers: SCENE_MANAGER.Draw(&m_Camera)
        Managers->>Scene: currentScene->Draw(camera)
        Scene->>GameObjects: DrawLayer(camera, WORLD)
        Scene->>GameObjects: DrawLayer(nullptr, UI)
        Game->>Renderer: DrawEnd()
    end
```

### 4.3 物理演算フロー

```mermaid
sequenceDiagram
    participant PhysicsManager
    participant Collider1
    participant Collider2
    participant GameObject1
    participant GameObject2
    
    PhysicsManager->>PhysicsManager: Update()
    PhysicsManager->>PhysicsManager: CleanupInvalidColliders()
    PhysicsManager->>PhysicsManager: CheckCollisions()
    
    loop For Each Collider Pair
        PhysicsManager->>PhysicsManager: ShouldCollide(col1, col2)
        PhysicsManager->>Collider1: CheckCollision(col2, info)
        Collider1->>Collider2: Collision Detection
        
        alt New Collision
            Collider1->>GameObject1: OnCollisionEnter(info)
            Collider2->>GameObject2: OnCollisionEnter(info)
        else Continuing Collision
            Collider1->>GameObject1: OnCollisionStay(info)
            Collider2->>GameObject2: OnCollisionStay(info)
        end
    end
    
    loop For Previous Collisions Not In Current
        PhysicsManager->>Collider1: OnCollisionExit(info)
        PhysicsManager->>Collider2: OnCollisionExit(info)
    end
```

### 4.4 リソース管理フロー

```mermaid
sequenceDiagram
    participant Component
    participant ResourceManager
    participant Cache
    participant Loader
    
    Component->>ResourceManager: LoadMesh("model.obj")
    ResourceManager->>Cache: Check m_MeshCache
    
    alt Cache Hit
        Cache-->>ResourceManager: Return cached mesh
        ResourceManager-->>Component: shared_ptr~StaticMesh~
    else Cache Miss
        ResourceManager->>Loader: Load mesh from file
        Loader->>Loader: Parse OBJ file
        Loader->>Loader: Load textures
        Loader-->>ResourceManager: New mesh object
        ResourceManager->>Cache: Store in m_MeshCache
        ResourceManager-->>Component: shared_ptr~StaticMesh~
    end
```

---

## 5. データフロー図

### 5.1 入力システムデータフロー

```mermaid
graph LR
    A[Hardware Input] --> B[Input Class]
    B --> C[IOManager]
    C --> D{Input Mode}
    D -->|Keyboard| E[Keyboard Mapping]
    D -->|Controller| F[Controller Mapping]
    E --> G[Game Logic]
    F --> G
    G --> H[GameObject Components]
    H --> I[Transform Update]
```

### 5.2 描画パイプライン

```mermaid
graph TB
    A[GameObject] --> B[Component::Draw]
    B --> C[MeshRendererComponent]
    C --> D[Set World Matrix]
    D --> E[Set Shader]
    E --> F[Set Material]
    F --> G[Set Texture]
    G --> H[Set Vertex/Index Buffer]
    H --> I[DrawIndexed]
    
    J[Camera] --> K[Set View Matrix]
    K --> L[Set Projection Matrix]
    L --> D
```

---

## 6. メモリ管理図

```mermaid
graph TB
    subgraph "Smart Pointer Management"
        A[unique_ptr] -->|Components| B[GameObject]
        C[shared_ptr] -->|Resources| D[ResourceManager]
        E[weak_ptr] -->|Optional| F[Cross-References]
    end
    
    subgraph "Object Lifetime"
        G[Scene Creation] --> H[GameObject Creation]
        H --> I[Component Attachment]
        I --> J[Resource Loading]
        J --> K[Update Loop]
        K --> L[Scene Destruction]
        L --> M[GameObject Cleanup]
        M --> N[Component Cleanup]
        N --> O[Resource Release]
    end
```

---

## 7. ディレクトリ構造

```
Project/
├── Core/
│   ├── Application.h/cpp      # ウィンドウ・メインループ管理
│   └── Game.h/cpp             # ゲームメインクラス
│
├── Manager/
│   ├── SceneManager.h/cpp     # シーン管理
│   ├── ResourceManager.h/cpp  # リソースキャッシュ管理
│   ├── PhysicsManager.h/cpp   # 物理演算・衝突判定管理
│   ├── SoundManager.h/cpp     # サウンド管理
│   ├── IOManager.h/cpp        # 入力管理
│   └── DataManager.h/cpp      # データ管理
│
├── GameObject/
│   ├── GameObject.h           # ゲームオブジェクト基底
│   ├── Transform.h            # 座標変換
│   ├── Component.h            # コンポーネント基底
│   ├── MeshRendererComponent.h
│   ├── PhysicsComponent/
│   │   ├── Collider.h/cpp
│   │   ├── SphereCollider.h/cpp
│   │   ├── AABBCollider.h/cpp
│   │   └── Rigidbody.h/cpp
│   └── TestComponent/         # テスト用コンポーネント
│
├── Graphics/
│   ├── Renderer.h/cpp         # 描画システム
│   ├── Shader.h/cpp           # シェーダー管理
│   ├── Texture.h/cpp          # テクスチャ管理
│   ├── VertexBuffer.h
│   ├── IndexBuffer.h
│   └── assimp/                # 3Dモデル読み込み
│       ├── AssimpParse.h/cpp
│       ├── StaticMesh.h/cpp
│       ├── Material.h
│       └── MeshRenderer.h
│
├── Scene/
│   ├── SceneBase.h/cpp        # シーン基底クラス
│   ├── Scene.h                # シーン定義
│   └── TestScene/             # テストシーン
│       ├── SceneTitle.h/cpp
│       ├── SceneGame.h/cpp
│       └── SceneResult.h/cpp
│
├── System/
│   ├── Camera.h/cpp           # カメラシステム
│   ├── input.h/cpp            # 入力システム
│   └── sound.h/cpp            # サウンドシステム
│
├── Util/
│   ├── SystemCommon.h         # 共通型定義
│   ├── PhysicsCommon.h        # 物理演算共通定義
│   ├── Timer.h/cpp            # タイマー
│   └── singleton.h/cpp        # シングルトン実装
│
└── main.h/cpp                 # エントリーポイント
```

---

## 8. 設計パターン使用状況

| パターン | 使用箇所 | 目的 |
|---------|---------|------|
| **Singleton** | 各種Manager | グローバルアクセス可能な単一インスタンス |
| **Component** | GameObject | 機能の組み合わせによる柔軟な設計 |
| **Factory** | ResourceManager | リソースの統一的な生成・管理 |
| **Observer** | Collision System | 衝突イベントの通知 |
| **State** | SceneManager | シーン遷移の管理 |
| **Template Method** | SceneBase | 共通処理の定義と個別実装の分離 |

---

## 9. 主要データ構造

### 9.1 頂点データ

```cpp
struct VERTEX_3D {
    DirectX::SimpleMath::Vector3 position;
    DirectX::SimpleMath::Vector3 normal;
    DirectX::SimpleMath::Color color;
    DirectX::SimpleMath::Vector2 uv;
};
```

### 9.2 衝突情報

```cpp
struct CollisionInfo {
    GameObject* other;
    Collider* otherCollider;
    Vector3 contactPoint;
    Vector3 contactNormal;
    float penetrationDepth;
    Vector3 relativeVelocity;
    float timestamp;
};
```

### 9.3 マテリアル

```cpp
struct MATERIAL {
    DirectX::SimpleMath::Color Ambient;
    DirectX::SimpleMath::Color Diffuse;
    DirectX::SimpleMath::Color Specular;
    DirectX::SimpleMath::Color Emission;
    float Shiness;
    BOOL TextureEnable;
};
```

---

## 10. スレッドセーフティ

| システム | スレッドセーフ | 備考 |
|---------|--------------|------|
| Singleton初期化 | ✅ Yes | std::call_once使用 |
| ResourceManager | ❌ No | 単一スレッド前提 |
| PhysicsManager | ❌ No | 単一スレッド前提 |
| 入力システム | ❌ No | 単一スレッド前提 |

**注意**: 現在は単一スレッドでの動作を前提としています。マルチスレッド化する場合は各マネージャーにミューテックスの追加が必要です。

---

## 11. パフォーマンス考慮事項

### キャッシュ戦略
- **ResourceManager**: ファイルパスをキーとしたリソースキャッシュ
- **ComponentMap**: type_indexによる高速なコンポーネント検索

### メモリプール
- 現在は標準のnew/deleteを使用
- 頻繁に生成・破棄されるオブジェクトには今後メモリプールの導入を検討

### 描画最適化
- サブセット単位での描画
- マテリアル・テクスチャのバッチング余地あり

---

このアーキテクチャドキュメントは、フレームワークの現状を詳細に記述しています。
次のセクションで、詳細な評価と改善提案を行います。
