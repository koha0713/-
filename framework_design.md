# DirectX ゲームフレームワーク 最終設計図

## 1. 全体構成図

```mermaid
graph TB
    subgraph "Application Layer"
        Main[main]
        App[Application]
        Game[Game]
    end
    
    subgraph "Scene Layer"
        SM[SceneManager<br/>Singleton]
        SB[SceneBase<br/>Abstract]
        ST[SceneTitle]
        SG[SceneGame]
    end
    
    subgraph "GameObject Layer"
        GO[GameObject]
        Comp[Component<br/>Abstract]
        Trans[Transform]
        MR[MeshRenderer]
        Col[Collider]
        RB[Rigidbody]
        PE[ParticleEmitter]
    end
    
    subgraph "System Layer"
        Ren[Renderer<br/>Static]
        IO[IOManager<br/>Singleton]
        Snd[Sound]
        CM[CollisionManager<br/>Singleton]
        Time[TimeManager<br/>Singleton]
    end
    
    subgraph "Resource Layer"
        RM[ResourceManager<T><br/>Template Singleton]
        Tex[Texture]
        Mod[Model]
        Mesh[Mesh]
        Mat[Material]
        Shd[Shader]
    end
    
    Main --> App
    App --> Game
    Game --> SM
    Game --> Ren
    
    SM --> SB
    SB --> ST
    SB --> SG
    SB --> GO
    
    GO --> Comp
    Comp --> Trans
    Comp --> MR
    Comp --> Col
    Comp --> RB
    Comp --> PE
    
    MR --> Mod
    MR --> Mat
    Mod --> Mesh
    Mat --> Tex
    Mat --> Shd
    
    GO --> CM
    Ren --> Shd
    
    style Main fill:#e1f5ff
    style App fill:#e1f5ff
    style Game fill:#e1f5ff
    style SM fill:#fff4e1
    style SB fill:#fff4e1
    style GO fill:#e8f5e9
    style Comp fill:#e8f5e9
    style Ren fill:#fce4ec
    style RM fill:#f3e5f5
```

## 2. コアシステム クラス図

```mermaid
classDiagram
    class Application {
        -HINSTANCE m_hInst
        -HWND m_hWnd
        -uint32_t m_Width
        -uint32_t m_Height
        +Application(width, height)
        +Run() void
        +GetWidth() uint32_t
        +GetHeight() uint32_t
        +GetWindow() HWND
        -InitApp() bool
        -UninitApp() void
        -MainLoop() void
        -WndProc() LRESULT
    }
    
    class Game {
        -Camera m_Camera
        +Game()
        +Init() void
        +Update() void
        +Draw() void
        +Uninit() void
    }
    
    class Renderer {
        -ID3D11Device* m_pDevice
        -ID3D11DeviceContext* m_pDeviceContext
        -IDXGISwapChain* m_pSwapChain
        -ID3D11RenderTargetView* m_pRenderTargetView
        -ID3D11DepthStencilView* m_pDepthStencilView
        +Init() HRESULT
        +Uninit() void
        +DrawStart() void
        +DrawEnd() void
        +SetWorldMatrix(Matrix*) void
        +SetViewMatrix(Matrix*) void
        +SetProjectionMatrix(Matrix*) void
        +GetDevice() ID3D11Device*
        +GetDeviceContext() ID3D11DeviceContext*
    }
    
    class Camera {
        -Vector3 m_Position
        -Vector3 m_Rotation
        -Vector3 m_Target
        -Matrix m_ViewMatrix
        -Matrix m_ProjectionMatrix
        +Init() void
        +Update() void
        +SetCamera() void
        +Uninit() void
        +GetViewMatrix() Matrix
        +GetProjectionMatrix() Matrix
    }
    
    Application --> Game
    Game --> Camera
    Game --> Renderer
```

## 3. シーン管理システム クラス図

```mermaid
classDiagram
    class SceneManager {
        -SCENE m_currentScene
        -unique_ptr~Scene~ m_scene
        +int score
        +Init() void
        +UnInit() void
        +Update() void
        +Draw() void
        +Draw(Camera*) void
        +ChangeScene(SCENE) void
    }
    
    class Scene {
        -unordered_map~SCENE, unique_ptr~SceneBase~~ m_sceneTable
        -SCENE m_startScene
        +Scene()
        +GetStartScene() SCENE
        +GetScene(SCENE) SceneBase*
    }
    
    class SceneBase {
        #bool m_isInitialized
        #bool m_isActive
        #vector~shared_ptr~GameObject~~ m_GameObjects
        #unique_ptr~Camera~ m_Camera
        +Init() void*
        +UnInit() void*
        +Update() void*
        +Draw() void*
        +AddGameObject~T~(string) T*
        +FindGameObject(string) GameObject*
        +FindGameObjectsWithTag(string) vector~GameObject*~
        +DestroyGameObject(GameObject*) void
        +IsInitialized() bool
        +IsActive() bool
    }
    
    class SceneTitle {
        +Init() void
        +UnInit() void
        +Update() void
        +Draw() void
    }
    
    class SceneGame {
        +Init() void
        +UnInit() void
        +Update() void
        +Draw() void
    }
    
    SceneManager --> Scene
    Scene --> SceneBase
    SceneBase <|-- SceneTitle
    SceneBase <|-- SceneGame
    SceneBase o-- GameObject
```

## 4. GameObjectシステム クラス図

```mermaid
classDiagram
    class GameObject {
        -string m_Name
        -string m_Tag
        -bool m_Active
        -Vector3 m_Position
        -Vector3 m_Rotation
        -Vector3 m_Scale
        -vector~shared_ptr~Component~~ m_Components
        +Init() void
        +Uninit() void
        +Update() void
        +Draw(Camera*) void
        +AddComponent~T~() T*
        +GetComponent~T~() T*
        +RemoveComponent~T~() void
        +SetActive(bool) void
        +IsActive() bool
        +SetName(string) void
        +GetName() string
        +SetTag(string) void
        +GetTag() string
    }
    
    class Component {
        #GameObject* m_Owner
        #bool m_Enabled
        +Init() void*
        +Uninit() void*
        +Update() void*
        +Draw(Camera*) void*
        +GetOwner() GameObject*
        +SetEnabled(bool) void
        +IsEnabled() bool
    }
    
    class Transform {
        -Vector3 m_LocalPosition
        -Vector3 m_LocalRotation
        -Vector3 m_LocalScale
        -Matrix m_WorldMatrix
        +Init() void
        +Update() void
        +GetWorldMatrix() Matrix
        +SetPosition(Vector3) void
        +GetPosition() Vector3
        +SetRotation(Vector3) void
        +GetRotation() Vector3
        +SetScale(Vector3) void
        +GetScale() Vector3
    }
    
    class MeshRenderer {
        -shared_ptr~Model~ m_Model
        -shared_ptr~Material~ m_Material
        +Init() void
        +Draw(Camera*) void
        +SetModel(shared_ptr~Model~) void
        +SetMaterial(shared_ptr~Material~) void
    }
    
    class Collider {
        <<abstract>>
        -ColliderType m_Type
        -bool m_IsTrigger
        +Intersects(Collider*) bool*
        +OnCollisionEnter(Collider*) void
        +OnCollisionStay(Collider*) void
        +OnCollisionExit(Collider*) void
    }
    
    class SphereCollider {
        -float m_Radius
        +Intersects(Collider*) bool
    }
    
    class BoxCollider {
        -Vector3 m_Size
        +Intersects(Collider*) bool
    }
    
    class Rigidbody {
        -Vector3 m_Velocity
        -Vector3 m_Acceleration
        -float m_Mass
        -bool m_UseGravity
        +Update() void
        +AddForce(Vector3) void
        +SetVelocity(Vector3) void
        +GetVelocity() Vector3
    }
    
    GameObject o-- Component
    Component <|-- Transform
    Component <|-- MeshRenderer
    Component <|-- Collider
    Component <|-- Rigidbody
    Collider <|-- SphereCollider
    Collider <|-- BoxCollider
```

## 5. グラフィックスシステム クラス図

```mermaid
classDiagram
    class Model {
        -vector~shared_ptr~Mesh~~ m_Meshes
        -string m_Directory
        +Load(string) bool
        +Draw() void
        +GetMeshCount() int
        +GetMesh(int) Mesh*
    }
    
    class Mesh {
        -VertexBuffer~VERTEX_3D~ m_VertexBuffer
        -IndexBuffer m_IndexBuffer
        -shared_ptr~Material~ m_Material
        -vector~VERTEX_3D~ m_Vertices
        -vector~unsigned int~ m_Indices
        +Create() void
        +Draw() void
        +SetMaterial(shared_ptr~Material~) void
    }
    
    class Material {
        -shared_ptr~Shader~ m_Shader
        -shared_ptr~Texture~ m_DiffuseMap
        -shared_ptr~Texture~ m_NormalMap
        -shared_ptr~Texture~ m_SpecularMap
        -Color m_Diffuse
        -Color m_Ambient
        -Color m_Specular
        -float m_Shininess
        +SetGPU() void
        +SetShader(shared_ptr~Shader~) void
        +SetTexture(TextureType, shared_ptr~Texture~) void
    }
    
    class Texture {
        -ComPtr~ID3D11ShaderResourceView~ m_SRV
        -ComPtr~ID3D11SamplerState~ m_SamplerState
        -int m_Width
        -int m_Height
        +Load(string) bool
        +SetGPU(int slot) void
        +Unload() void
        +GetWidth() int
        +GetHeight() int
    }
    
    class Shader {
        -ComPtr~ID3D11VertexShader~ m_pVertexShader
        -ComPtr~ID3D11PixelShader~ m_pPixelShader
        -ComPtr~ID3D11InputLayout~ m_pVertexLayout
        +Create(string vs, string ps) void
        +SetGPU() void
    }
    
    class VertexBuffer~T~ {
        -ComPtr~ID3D11Buffer~ m_VertexBuffer
        +Create(vector~T~) void
        +SetGPU() void
        +Modify(vector~T~) void
    }
    
    class IndexBuffer {
        -ComPtr~ID3D11Buffer~ m_IndexBuffer
        +Create(vector~unsigned int~) void
        +SetGPU() void
    }
    
    Model o-- Mesh
    Mesh --> Material
    Mesh --> VertexBuffer
    Mesh --> IndexBuffer
    Material --> Shader
    Material --> Texture
```

## 6. エフェクトシステム クラス図

```mermaid
classDiagram
    class Particle {
        +Vector3 m_Position
        +Vector3 m_Velocity
        +Vector3 m_Acceleration
        +Color m_Color
        +float m_LifeTime
        +float m_CurrentTime
        +float m_Size
        +float m_Rotation
        +Update(float deltaTime) void
        +IsAlive() bool
    }
    
    class ParticleEmitter {
        -vector~Particle~ m_Particles
        -shared_ptr~Texture~ m_ParticleTexture
        -int m_MaxParticles
        -float m_EmissionRate
        -float m_EmissionTimer
        -VertexBuffer~VERTEX_3D~ m_VertexBuffer
        -bool m_Loop
        +Init() void
        +Update() void
        +Draw(Camera*) void
        +Emit(int count) void
        +SetTexture(shared_ptr~Texture~) void
        +SetEmissionRate(float) void
        +SetMaxParticles(int) void
    }
    
    class PostEffect {
        <<abstract>>
        #ComPtr~ID3D11Texture2D~ m_RenderTarget
        #ComPtr~ID3D11RenderTargetView~ m_RTV
        #ComPtr~ID3D11ShaderResourceView~ m_SRV
        #shared_ptr~Shader~ m_Shader
        +Init() void*
        +Apply() void*
        +GetResultTexture() ID3D11ShaderResourceView*
    }
    
    class BloomEffect {
        -float m_Threshold
        -float m_Intensity
        +Init() void
        +Apply() void
        +SetThreshold(float) void
        +SetIntensity(float) void
    }
    
    class BlurEffect {
        -int m_BlurRadius
        +Init() void
        +Apply() void
        +SetBlurRadius(int) void
    }
    
    class ToneMappingEffect {
        -float m_Exposure
        +Init() void
        +Apply() void
        +SetExposure(float) void
    }
    
    ParticleEmitter o-- Particle
    Component <|-- ParticleEmitter
    PostEffect <|-- BloomEffect
    PostEffect <|-- BlurEffect
    PostEffect <|-- ToneMappingEffect
```

## 7. リソース管理システム クラス図

```mermaid
classDiagram
    class ResourceManager~T~ {
        -unordered_map~string, shared_ptr~T~~ m_Resources
        -mutex m_Mutex
        +Load(string path) shared_ptr~T~
        +Unload(string path) void
        +UnloadAll() void
        +Get(string path) shared_ptr~T~
        +IsLoaded(string path) bool
        +GetResourceCount() int
    }
    
    class TextureManager {
        <<singleton>>
    }
    
    class ModelManager {
        <<singleton>>
    }
    
    class ShaderManager {
        <<singleton>>
    }
    
    class AudioManager {
        <<singleton>>
    }
    
    ResourceManager <|-- TextureManager : instantiation~Texture~
    ResourceManager <|-- ModelManager : instantiation~Model~
    ResourceManager <|-- ShaderManager : instantiation~Shader~
    ResourceManager <|-- AudioManager : instantiation~AudioClip~
    
    TextureManager --> Texture : manages
    ModelManager --> Model : manages
    ShaderManager --> Shader : manages
```

## 8. 物理・衝突判定システム クラス図

```mermaid
classDiagram
    class CollisionManager {
        <<singleton>>
        -vector~Collider*~ m_Colliders
        -vector~CollisionPair~ m_PreviousCollisions
        +AddCollider(Collider*) void
        +RemoveCollider(Collider*) void
        +CheckCollisions() void
        +Update() void
        -CheckCollisionPair(Collider*, Collider*) bool
        -ProcessCollisionEvents() void
    }
    
    class CollisionPair {
        +Collider* colliderA
        +Collider* colliderB
        +CollisionPair(Collider*, Collider*)
        +operator==(CollisionPair) bool
    }
    
    class PhysicsWorld {
        <<singleton>>
        -Vector3 m_Gravity
        -vector~Rigidbody*~ m_Rigidbodies
        +SetGravity(Vector3) void
        +GetGravity() Vector3
        +AddRigidbody(Rigidbody*) void
        +RemoveRigidbody(Rigidbody*) void
        +Update(float deltaTime) void
        +Raycast(Vector3 origin, Vector3 direction, float distance) RaycastHit
    }
    
    class RaycastHit {
        +bool hit
        +GameObject* object
        +Vector3 point
        +Vector3 normal
        +float distance
    }
    
    CollisionManager o-- CollisionPair
    CollisionManager --> Collider : manages
    PhysicsWorld --> Rigidbody : manages
    PhysicsWorld ..> RaycastHit : creates
```

## 9. UIシステム クラス図

```mermaid
classDiagram
    class UIElement {
        <<abstract>>
        #Vector2 m_AnchorMin
        #Vector2 m_AnchorMax
        #Vector2 m_Pivot
        #Vector2 m_Position
        #Vector2 m_Size
        #bool m_Visible
        +Init() void*
        +Update() void*
        +Draw() void*
        +SetAnchor(Vector2, Vector2) void
        +SetPivot(Vector2) void
        +SetVisible(bool) void
        +IsVisible() bool
        +Contains(Vector2) bool
    }
    
    class UIText {
        -string m_Text
        -shared_ptr~Font~ m_Font
        -Color m_Color
        -int m_FontSize
        -TextAlign m_Alignment
        +Init() void
        +Draw() void
        +SetText(string) void
        +SetFont(shared_ptr~Font~) void
        +SetColor(Color) void
        +SetFontSize(int) void
    }
    
    class UIImage {
        -shared_ptr~Texture~ m_Texture
        -Color m_TintColor
        -VertexBuffer~VERTEX_3D~ m_VertexBuffer
        +Init() void
        +Draw() void
        +SetTexture(shared_ptr~Texture~) void
        +SetTintColor(Color) void
    }
    
    class UIButton {
        -shared_ptr~UIImage~ m_Background
        -shared_ptr~UIText~ m_Label
        -function~void()~ m_OnClick
        -bool m_IsHovered
        -bool m_IsPressed
        +Init() void
        +Update() void
        +Draw() void
        +SetOnClick(function~void()~) void
        -CheckHover() void
        -CheckClick() void
    }
    
    class UICanvas {
        -vector~shared_ptr~UIElement~~ m_Elements
        -CanvasScaleMode m_ScaleMode
        +AddElement(shared_ptr~UIElement~) void
        +RemoveElement(UIElement*) void
        +Update() void
        +Draw() void
    }
    
    GameObject <|-- UIElement
    UIElement <|-- UIText
    UIElement <|-- UIImage
    UIElement <|-- UIButton
    UIButton o-- UIImage
    UIButton o-- UIText
    UICanvas o-- UIElement
```

## 10. 入力システム クラス図

```mermaid
classDiagram
    class Input {
        -BYTE keyState[256]
        -BYTE keyState_old[256]
        -XINPUT_STATE controllerState
        -XINPUT_STATE controllerState_old
        -int VibrationTime
        +Input()
        +Update() void
        +GetKeyPress(int) bool
        +GetKeyTrigger(int) bool
        +GetKeyRelease(int) bool
        +GetLeftAnalogStick() XMFLOAT2
        +GetRightAnalogStick() XMFLOAT2
        +GetLeftTrigger() float
        +GetRightTrigger() float
        +GetButtonPress(WORD) bool
        +GetButtonTrigger(WORD) bool
        +GetButtonRelease(WORD) bool
        +SetVibration(int, float) void
    }
    
    class IOManager {
        <<singleton>>
        -INPUT_MODE m_mode
        -map~INPUT_MODE, map~INPUT_TYPE, int~~ m_keyBindTable
        -Input input
        +Init() void
        +UnInit() void
        +Update() void
        +SetInputMode(INPUT_MODE) void
        +GetInputMode() INPUT_MODE
        +GetKeyDown(INPUT_TYPE) bool
        +GetKeyPress(INPUT_TYPE) bool
        +GetKeyUp(INPUT_TYPE) bool
    }
    
    IOManager o-- Input
```

## 11. サウンドシステム クラス図

```mermaid
classDiagram
    class Sound {
        -IXAudio2* m_pXAudio2
        -IXAudio2MasteringVoice* m_pMasteringVoice
        -IXAudio2SourceVoice* m_pSourceVoice[]
        -WAVEFORMATEXTENSIBLE m_wfx[]
        -XAUDIO2_BUFFER m_buffer[]
        -BYTE* m_DataBuffer[]
        +Init() HRESULT
        +Uninit() void
        +Play(SOUND_LABEL) void
        +Stop(SOUND_LABEL) void
        +Resume(SOUND_LABEL) void
        -FindChunk() HRESULT
        -ReadChunkData() HRESULT
    }
    
    class AudioClip {
        -IXAudio2SourceVoice* m_pSourceVoice
        -WAVEFORMATEXTENSIBLE m_wfx
        -XAUDIO2_BUFFER m_buffer
        -BYTE* m_DataBuffer
        -string m_FilePath
        +Load(string) bool
        +Play() void
        +Stop() void
        +Pause() void
        +Resume() void
        +SetVolume(float) void
        +SetLoop(bool) void
    }
    
    class AudioSource {
        -shared_ptr~AudioClip~ m_Clip
        -float m_Volume
        -bool m_Loop
        -bool m_IsPlaying
        +Init() void
        +Update() void
        +Play() void
        +Stop() void
        +Pause() void
        +SetClip(shared_ptr~AudioClip~) void
        +SetVolume(float) void
    }
    
    Component <|-- AudioSource
    AudioSource --> AudioClip
```

## 12. ユーティリティクラス図

```mermaid
classDiagram
    class Singleton~T~ {
        -static once_flag initFlag
        -static T* s_instance
        +GetInstance() T&
        -CreateInstance() void
        -DeleteInstance() void
    }
    
    class SingletonFinalizer {
        +addFinalizer(FinalizerFunc) void
        +finalize() void
    }
    
    class TimeManager {
        <<singleton>>
        -float m_DeltaTime
        -float m_TimeScale
        -long long m_FrameCount
        +Update() void
        +GetDeltaTime() float
        +GetTimeScale() float
        +SetTimeScale(float) void
        +GetFrameCount() long long
    }
    
    class MathUtility {
        +Lerp(float, float, float) float
        +Clamp(float, float, float) float
        +Distance(Vector3, Vector3) float
        +Normalize(Vector3) Vector3
        +Cross(Vector3, Vector3) Vector3
        +Dot(Vector3, Vector3) float
    }
    
    Singleton <.. SingletonFinalizer : uses
```

## 13. システム依存関係図

```mermaid
graph LR
    subgraph "High Level"
        A[GameObject System]
        B[Scene System]
    end
    
    subgraph "Mid Level"
        C[Graphics System]
        D[Physics System]
        E[Effect System]
        F[UI System]
    end
    
    subgraph "Low Level"
        G[Resource Manager]
        H[Renderer]
        I[Input Manager]
        J[Sound System]
    end
    
    A --> C
    A --> D
    A --> E
    B --> A
    C --> G
    C --> H
    D --> H
    E --> C
    F --> H
    F --> G
    
    style A fill:#e8f5e9
    style B fill:#fff4e1
    style C fill:#e3f2fd
    style D fill:#fce4ec
    style E fill:#f3e5f5
    style F fill:#fff9c4
    style G fill:#ffebee
    style H fill:#e0f2f1
    style I fill:#fce4ec
    style J fill:#e8eaf6
```

