# DirectX ゲームフレームワーク 改善版設計図（修正版）

## 1. 全体アーキテクチャ概要

```mermaid
graph TB
    subgraph "Application Layer"
        Main[main]
        App[Application]
        Game[Game]
    end
    
    subgraph "Scene Management"
        SM[SceneManager<br/>Singleton]
        SB[IScene<br/>Interface]
        ST[SceneTitle]
        SG[SceneGame]
    end
    
    subgraph "GameObject System"
        GO[GameObject]
        IU[IUpdatable<br/>Interface]
        ID[IDrawable<br/>Interface]
        Comp[Component<br/>Abstract]
    end
    
    subgraph "Core Systems"
        Ren[Renderer<br/>Static]
        CM[CollisionManager<br/>Singleton]
        Time[TimeManager<br/>Singleton]
        Debug[DebugRenderer<br/>Singleton]
    end
    
    subgraph "Resource Management"
        RM[ResourceManager<br/>Template Singleton]
        AL[AsyncLoader]
        RC[ResourceCache]
    end
    
    Main --> App
    App --> Game
    Game --> SM
    SM --> SB
    SB -->|implements| IU
    SB -->|implements| ID
    GO -->|implements| IU
    GO -->|implements| ID
    Comp -->|implements| IU
    Comp -->|implements| ID
    
    style Main fill:#e1f5ff
    style SM fill:#fff4e1
    style GO fill:#e8f5e9
    style Ren fill:#fce4ec
    style RM fill:#f3e5f5
    style IU fill:#f0f0f0
    style ID fill:#f0f0f0
```

## 2. コアシステム・インターフェース設計

```mermaid
classDiagram
    class IUpdatable {
        <<interface>>
        +Update(float deltaTime) void*
        +IsActive() bool*
    }
    
    class IDrawable {
        <<interface>>
        +Draw(Camera* camera) void*
        +IsVisible() bool*
        +GetRenderPriority() int*
    }
    
    class IScene {
        <<interface>>
        +Init() Result~void~*
        +Uninit() void*
        +Update(float deltaTime) void*
        +Draw() void*
        +OnEnter() void*
        +OnExit() void*
        +OnPause() void*
        +OnResume() void*
    }
    
    class Result~T~ {
        -optional~T~ m_value
        -string m_error
        +IsSuccess() bool
        +GetValue() T&
        +GetError() string
        +Success(T value) Result~T~$
        +Error(string error) Result~T~$
    }
    
    class Application {
        -HINSTANCE m_hInst
        -HWND m_hWnd
        -uint32_t m_Width
        -uint32_t m_Height
        -unique_ptr~Game~ m_Game
        +Application(width, height)
        +Run() Result~void~
        +GetWindow() HWND
        -InitApp() Result~void~
        -UninitApp() void
        -MainLoop() void
        -WndProc() LRESULT
    }
    
    class Game {
        -unique_ptr~Camera~ m_MainCamera
        -unique_ptr~DebugRenderer~ m_DebugRenderer
        +Init() Result~void~
        +Update(float deltaTime) void
        +Draw() void
        +Uninit() void
        +GetMainCamera() Camera*
    }
    
    Application --> Game
    Application --> Result : uses
    Game -->|implements| IUpdatable
    Game -->|implements| IDrawable
```

## 3. メモリ管理統一版 GameObject システム

```mermaid
classDiagram
    class GameObject {
        -string m_Name
        -string m_Tag
        -uint32_t m_Layer
        -bool m_Active
        -unique_ptr~Transform~ m_Transform
        -vector~unique_ptr~Component~~ m_Components
        -weak_ptr~GameObject~ m_Parent
        -vector~shared_ptr~GameObject~~ m_Children
        +Init() Result~void~
        +Uninit() void
        +Update(float deltaTime) void
        +Draw(Camera* camera) void
        +LateUpdate(float deltaTime) void
        +FixedUpdate(float fixedDeltaTime) void
        +AddComponent~T~() Result~T*~
        +GetComponent~T~() T*
        +RemoveComponent~T~() void
        +GetTransform() Transform*
        +SetParent(weak_ptr~GameObject~) void
        +AddChild(shared_ptr~GameObject~) void
        +FindChild(string name) shared_ptr~GameObject~
    }
    
    class Transform {
        -Vector3 m_LocalPosition
        -Quaternion m_LocalRotation
        -Vector3 m_LocalScale
        -Matrix m_WorldMatrix
        -bool m_IsDirty
        +UpdateWorldMatrix() void
        +GetWorldMatrix() Matrix
        +SetLocalPosition(Vector3) void
        +GetWorldPosition() Vector3
        +TransformPoint(Vector3) Vector3
        +InverseTransformPoint(Vector3) Vector3
    }
    
    class Component {
        <<abstract>>
        #weak_ptr~GameObject~ m_Owner
        #bool m_Enabled
        +Init() Result~void~*
        +Uninit() void*
        +Update(float deltaTime) void*
        +LateUpdate(float deltaTime) void*
        +FixedUpdate(float fixedDeltaTime) void*
        +Draw(Camera* camera) void*
        +OnEnable() void*
        +OnDisable() void*
        +GetOwner() GameObject*
        +GetTransform() Transform*
        +SetEnabled(bool) void
        +IsEnabled() bool
    }
    
    class MeshRenderer {
        -shared_ptr~Model~ m_Model
        -shared_ptr~Material~ m_Material
        -bool m_CastShadows
        -bool m_ReceiveShadows
        +Init() Result~void~
        +Draw(Camera* camera) void
        +SetModel(shared_ptr~Model~) void
        +SetMaterial(shared_ptr~Material~) void
        +GetBounds() BoundingBox
    }
    
    GameObject -->|implements| IUpdatable
    GameObject -->|implements| IDrawable
    GameObject *-- Transform
    GameObject o-- Component
    Component -->|implements| IUpdatable
    Component -->|implements| IDrawable
    Component <|-- MeshRenderer
```

## 4. 改善版シーン管理システム

```mermaid
classDiagram
    class SceneManager {
        <<singleton>>
        -unique_ptr~SceneBase~ m_CurrentScene
        -unique_ptr~SceneBase~ m_NextScene
        -unique_ptr~SceneBase~ m_LoadingScene
        -bool m_IsTransitioning
        -float m_TransitionProgress
        -SceneTransition m_TransitionType
        +Init() Result~void~
        +Update(float deltaTime) void
        +Draw() void
        +ChangeScene~T~(SceneTransition type) Result~void~
        +ChangeSceneAsync~T~() future~Result~void~~
        +PushScene~T~() void
        +PopScene() void
        +GetCurrentScene() SceneBase*
        -HandleTransition() void
    }
    
    class SceneBase {
        <<abstract>>
        #bool m_IsInitialized
        #bool m_IsActive
        #bool m_IsPaused
        #vector~shared_ptr~GameObject~~ m_GameObjects
        #vector~shared_ptr~GameObject~~ m_UIObjects
        #unique_ptr~Camera~ m_SceneCamera
        #unique_ptr~UICanvas~ m_UICanvas
        #unordered_map~string, shared_ptr~GameObject~~ m_NamedObjects
        +Init() Result~void~*
        +Uninit() void*
        +Update(float deltaTime) void*
        +Draw() void*
        +OnEnter() void*
        +OnExit() void*
        +OnPause() void*
        +OnResume() void*
        +CreateGameObject(string name) shared_ptr~GameObject~
        +FindGameObject(string name) weak_ptr~GameObject~
        +FindGameObjectsWithTag(string tag) vector~weak_ptr~GameObject~~
        +DestroyGameObject(shared_ptr~GameObject~) void
        +PreloadResources() future~void~*
    }
    
    class SceneTransition {
        <<enumeration>>
        None
        Fade
        Slide
        Zoom
        Custom
    }
    
    SceneManager --> SceneBase
    SceneBase -->|implements| IScene
    SceneManager --> SceneTransition
```

## 5. 最適化版コリジョンシステム

```mermaid
classDiagram
    class CollisionManager {
        <<singleton>>
        -unique_ptr~SpatialPartition~ m_SpatialPartition
        -vector~weak_ptr~Collider~~ m_StaticColliders
        -vector~weak_ptr~Collider~~ m_DynamicColliders
        -unordered_map~CollisionPair, CollisionInfo~ m_CollisionCache
        -CollisionMatrix m_LayerCollisionMatrix
        +Init() Result~void~
        +Update(float deltaTime) void
        +AddCollider(weak_ptr~Collider~, bool isStatic) void
        +RemoveCollider(Collider*) void
        +CheckCollisions() void
        +Raycast(Ray ray, float maxDistance) RaycastHit
        +RaycastAll(Ray ray, float maxDistance) vector~RaycastHit~
        +OverlapSphere(Vector3 center, float radius) vector~weak_ptr~Collider~~
        +SetLayerCollision(uint32_t layer1, uint32_t layer2, bool enable) void
        -BroadPhase() vector~CollisionPair~
        -NarrowPhase(CollisionPair pair) optional~CollisionInfo~
    }
    
    class SpatialPartition {
        <<abstract>>
        +Insert(weak_ptr~Collider~) void*
        +Remove(Collider*) void*
        +Update(weak_ptr~Collider~) void*
        +Query(BoundingBox region) vector~weak_ptr~Collider~~*
        +Clear() void*
    }
    
    class QuadTree {
        -unique_ptr~QuadNode~ m_Root
        -int m_MaxDepth
        -int m_MaxObjectsPerNode
        +Insert(weak_ptr~Collider~) void
        +Query(BoundingBox region) vector~weak_ptr~Collider~~
        -Subdivide(QuadNode* node) void
    }
    
    class OctTree {
        -unique_ptr~OctNode~ m_Root
        -int m_MaxDepth
        -int m_MaxObjectsPerNode
        +Insert(weak_ptr~Collider~) void
        +Query(BoundingBox region) vector~weak_ptr~Collider~~
        -Subdivide(OctNode* node) void
    }
    
    class Collider {
        <<abstract>>
        #ColliderType m_Type
        #uint32_t m_Layer
        #bool m_IsTrigger
        #bool m_IsStatic
        #weak_ptr~GameObject~ m_GameObject
        #PhysicsMaterial m_Material
        #BoundingBox m_Bounds
        +Intersects(Collider* other) bool*
        +GetBounds() BoundingBox
        +OnCollisionEnter(CollisionInfo info) void
        +OnCollisionStay(CollisionInfo info) void
        +OnCollisionExit(weak_ptr~Collider~) void
        +OnTriggerEnter(weak_ptr~Collider~) void
        +OnTriggerStay(weak_ptr~Collider~) void
        +OnTriggerExit(weak_ptr~Collider~) void
    }
    
    class CollisionInfo {
        +weak_ptr~Collider~ otherCollider
        +Vector3 contactPoint
        +Vector3 contactNormal
        +float penetrationDepth
        +Vector3 relativeVelocity
    }
    
    CollisionManager --> SpatialPartition
    SpatialPartition <|-- QuadTree
    SpatialPartition <|-- OctTree
    CollisionManager --> Collider
    Collider --> CollisionInfo
```

## 6. レンダリング最適化システム

```mermaid
classDiagram
    class Renderer {
        <<static>>
        -ID3D11Device* s_pDevice
        -ID3D11DeviceContext* s_pDeviceContext
        -unique_ptr~RenderCommandBuffer~ s_CommandBuffer
        -unique_ptr~RenderStateCache~ s_StateCache
        -vector~RenderQueue~ s_RenderQueues
        +Init(HWND hwnd) Result~void~
        +BeginFrame() void
        +EndFrame() void
        +Submit(RenderCommand command) void
        +ExecuteCommands() void
        +SetRenderTarget(RenderTarget* target) void
        +SetViewport(Viewport viewport) void
        +EnableBatching(bool enable) void
        +GetDevice() ID3D11Device*
        +GetContext() ID3D11DeviceContext*
    }
    
    class RenderCommandBuffer {
        -vector~RenderCommand~ m_Commands
        -unordered_map~uint64_t, RenderBatch~ m_Batches
        +AddCommand(RenderCommand cmd) void
        +Sort() void
        +Execute() void
        +Clear() void
        +EnableBatching(bool enable) void
        -BatchCommands() void
        -GenerateSortKey(RenderCommand cmd) uint64_t
    }
    
    class RenderCommand {
        +CommandType type
        +shared_ptr~Mesh~ mesh
        +shared_ptr~Material~ material
        +Matrix worldMatrix
        +uint32_t renderQueue
        +float distanceToCamera
        +RenderStateOverride stateOverride
    }
    
    class RenderQueue {
        <<enumeration>>
        Background = 1000
        Geometry = 2000
        AlphaTest = 2450
        Transparent = 3000
        Overlay = 4000
        UI = 5000
    }
    
    class RenderBatch {
        +shared_ptr~Material~ material
        +vector~Matrix~ worldMatrices
        +shared_ptr~VertexBuffer~ instanceBuffer
        +uint32_t instanceCount
        +Draw() void
    }
    
    class FrustumCuller {
        -Frustum m_ViewFrustum
        +SetViewProjectionMatrix(Matrix viewProj) void
        +IsVisible(BoundingBox bounds) bool
        +CullObjects(vector~GameObject*~) vector~GameObject*~
    }
    
    Renderer --> RenderCommandBuffer
    RenderCommandBuffer o-- RenderCommand
    RenderCommandBuffer --> RenderBatch
    Renderer --> FrustumCuller
```

## 7. 非同期リソース管理システム

```mermaid
classDiagram
    class ResourceManager~T~ {
        <<template singleton>>
        -unordered_map~string, shared_ptr~T~~ m_Resources
        -unordered_map~string, weak_ptr~T~~ m_WeakCache
        -unique_ptr~AsyncLoader~ m_AsyncLoader
        -unique_ptr~ResourceCache~ m_Cache
        +Load(string path) Result~shared_ptr~T~~
        +LoadAsync(string path) future~Result~shared_ptr~T~~~
        +Unload(string path) void
        +UnloadUnused() void
        +Get(string path) shared_ptr~T~
        +IsLoaded(string path) bool
        +PreloadResources(vector~string~ paths) future~void~
        +SetCacheSize(size_t bytes) void
        +GetMemoryUsage() size_t
        -LoadFromFile(string path) Result~shared_ptr~T~~
        -CreateResource(string path, byte* data) Result~shared_ptr~T~~
    }
    
    class AsyncLoader {
        -ThreadPool m_ThreadPool
        -queue~LoadTask~ m_LoadQueue
        -mutex m_QueueMutex
        +LoadAsync~T~(string path) future~Result~T~~
        +CancelPendingLoads() void
        +GetPendingCount() int
        -ProcessLoadQueue() void
    }
    
    class ResourceCache {
        -size_t m_MaxCacheSize
        -size_t m_CurrentSize
        -list~CacheEntry~ m_LRUList
        -unordered_map~string, list~CacheEntry~::iterator~ m_CacheMap
        +Add(string key, shared_ptr~void~ resource, size_t size) void
        +Get(string key) shared_ptr~void~
        +Remove(string key) void
        +Clear() void
        +SetMaxSize(size_t bytes) void
        -EvictLRU() void
    }
    
    class ThreadPool {
        -vector~thread~ m_Workers
        -queue~function~void()~~ m_Tasks
        -mutex m_QueueMutex
        -condition_variable m_Condition
        -bool m_Stop
        +ThreadPool(size_t numThreads)
        +Submit(function~void()~ task) future~void~
        +Shutdown() void
        -WorkerThread() void
    }
    
    ResourceManager --> AsyncLoader
    ResourceManager --> ResourceCache
    AsyncLoader --> ThreadPool
```

## 8. エラーハンドリング・デバッグシステム

```mermaid
classDiagram
    class ErrorHandler {
        <<singleton>>
        -vector~ErrorCallback~ m_ErrorCallbacks
        -ofstream m_LogFile
        -ErrorLevel m_MinLogLevel
        +LogError(ErrorLevel level, string message) void
        +LogWarning(string message) void
        +LogInfo(string message) void
        +RegisterCallback(ErrorCallback callback) void
        +SetMinLogLevel(ErrorLevel level) void
        +Assert(bool condition, string message) void
        +GetLastError() optional~string~
    }
    
    class DebugRenderer {
        <<singleton>>
        -vector~DebugLine~ m_Lines
        -vector~DebugBox~ m_Boxes
        -vector~DebugSphere~ m_Spheres
        -vector~DebugText~ m_Texts
        -shared_ptr~Shader~ m_DebugShader
        -unique_ptr~VertexBuffer~ m_LineBuffer
        +DrawLine(Vector3 start, Vector3 end, Color color, float duration) void
        +DrawBox(BoundingBox box, Color color, float duration) void
        +DrawSphere(Vector3 center, float radius, Color color, float duration) void
        +DrawFrustum(Frustum frustum, Color color, float duration) void
        +DrawText(Vector3 position, string text, Color color, float duration) void
        +DrawGrid(float size, int divisions, Color color) void
        +Update(float deltaTime) void
        +Render(Camera* camera) void
        +Clear() void
        +SetEnabled(bool enabled) void
    }
    
    class Profiler {
        <<singleton>>
        -unordered_map~string, ProfileBlock~ m_Blocks
        -stack~ProfileTimer~ m_TimerStack
        +BeginBlock(string name) void
        +EndBlock() void
        +GetAverageTime(string block) float
        +GetMaxTime(string block) float
        +Reset() void
        +GenerateReport() string
    }
    
    class ProfileTimer {
        -string m_Name
        -chrono::high_resolution_clock::time_point m_StartTime
        +ProfileTimer(string name)
        +~ProfileTimer()
        +GetElapsedTime() float
    }
    
    ErrorHandler --> DebugRenderer
    Profiler --> ProfileTimer
```

## 9. 改善版パーティクルシステム

```mermaid
classDiagram
    class ParticleSystem {
        -vector~ParticleEmitter*~ m_Emitters
        -unique_ptr~ComputeShader~ m_UpdateShader
        -bool m_UseGPUSimulation
        +Init() Result~void~
        +Update(float deltaTime) void
        +Render(Camera* camera) void
        +AddEmitter(ParticleEmitter* emitter) void
        +SetGPUSimulation(bool enable) void
    }
    
    class ParticleEmitter {
        -ParticlePool m_ParticlePool
        -ParticleEmitterSettings m_Settings
        -Vector3 m_Position
        -float m_EmissionRate
        -float m_EmissionTimer
        -bool m_IsEmitting
        +Init(ParticleEmitterSettings settings) Result~void~
        +Update(float deltaTime) void
        +Emit(int count) void
        +Start() void
        +Stop() void
        +SetPosition(Vector3 pos) void
        +SetEmissionRate(float rate) void
        +GetActiveParticleCount() int
    }
    
    class ParticlePool {
        -vector~Particle~ m_Particles
        -queue~int~ m_FreeIndices
        -int m_ActiveCount
        -int m_MaxParticles
        +Init(int maxParticles) void
        +Spawn() Particle*
        +Despawn(int index) void
        +GetActiveParticles() vector~Particle*~
        +Clear() void
    }
    
    class Particle {
        +Vector3 position
        +Vector3 velocity
        +Vector3 acceleration
        +Color color
        +float size
        +float lifetime
        +float age
        +float rotation
        +float rotationSpeed
        +bool isActive
    }
    
    class GPUParticleBuffer {
        -ID3D11Buffer* m_ParticleBuffer
        -ID3D11ShaderResourceView* m_SRV
        -ID3D11UnorderedAccessView* m_UAV
        +Create(int maxParticles) Result~void~
        +Update(ComputeShader* shader) void
        +GetSRV() ID3D11ShaderResourceView*
    }
    
    ParticleSystem --> ParticleEmitter
    ParticleEmitter --> ParticlePool
    ParticlePool o-- Particle
    ParticleSystem --> GPUParticleBuffer
```

## 10. 改善版UIシステム

```mermaid
classDiagram
    class UICanvas {
        -vector~shared_ptr~UIElement~~ m_Elements
        -CanvasScaleMode m_ScaleMode
        -Vector2 m_ReferenceResolution
        -RenderTexture m_RenderTarget
        -bool m_WorldSpace
        +AddElement(shared_ptr~UIElement~) void
        +RemoveElement(UIElement*) void
        +Update(float deltaTime) void
        +Draw() void
        +HandleInput(InputEvent event) bool
        +FindElement(string name) weak_ptr~UIElement~
        +SetScaleMode(CanvasScaleMode mode) void
        +ConvertScreenToCanvas(Vector2 screen) Vector2
    }
    
    class UIElement {
        <<abstract>>
        #string m_Name
        #RectTransform m_RectTransform
        #bool m_Visible
        #bool m_Interactable
        #int m_RenderOrder
        #weak_ptr~UIElement~ m_Parent
        #vector~shared_ptr~UIElement~~ m_Children
        +Init() Result~void~*
        +Update(float deltaTime) void*
        +Draw() void*
        +OnPointerEnter() void*
        +OnPointerExit() void*
        +OnPointerDown() void*
        +OnPointerUp() void*
        +GetWorldRect() Rect
        +Contains(Vector2 point) bool
        +SetParent(weak_ptr~UIElement~) void
        +AddChild(shared_ptr~UIElement~) void
    }
    
    class RectTransform {
        +Vector2 anchorMin
        +Vector2 anchorMax
        +Vector2 pivot
        +Vector2 anchoredPosition
        +Vector2 sizeDelta
        +Vector3 localScale
        +float rotation
        +GetRect() Rect
        +GetWorldMatrix() Matrix
    }
    
    class UIButton {
        -shared_ptr~UIImage~ m_Background
        -shared_ptr~UIText~ m_Label
        -function~void()~ m_OnClick
        -ButtonState m_State
        -UIButtonStyle m_Style
        +Init() Result~void~
        +Update(float deltaTime) void
        +Draw() void
        +SetOnClick(function~void()~) void
        +SetStyle(UIButtonStyle style) void
        +SetInteractable(bool interactable) void
    }
    
    class UIImage {
        -shared_ptr~Texture~ m_Texture
        -Color m_TintColor
        +SetTexture(shared_ptr~Texture~) void
        +SetTintColor(Color) void
    }
    
    class UIText {
        -string m_Text
        -shared_ptr~Font~ m_Font
        -Color m_Color
        +SetText(string) void
        +SetFont(shared_ptr~Font~) void
        +SetColor(Color) void
    }
    
    UICanvas o-- UIElement
    UIElement *-- RectTransform
    UIElement <|-- UIButton
    UIElement <|-- UIImage
    UIElement <|-- UIText
    UIButton --> UIImage
    UIButton --> UIText
```

## 11. 統合システム依存関係図

```mermaid
graph TB
    subgraph "Application Layer"
        APP[Application]
        GAME[Game]
    end
    
    subgraph "High Level Systems"
        SCENE[Scene System<br/>+ Async Transitions]
        GO[GameObject System<br/>+ Component Model]
    end
    
    subgraph "Rendering Pipeline"
        RENDERER[Renderer<br/>+ Command Buffer]
        BATCH[Batch System]
        FRUSTUM[Frustum Culling]
        DEBUG[Debug Renderer]
    end
    
    subgraph "Physics & Collision"
        PHYSICS[Physics System]
        COLLISION[Collision Manager<br/>+ Spatial Partition]
        RAYCAST[Raycast System]
    end
    
    subgraph "Resource Management"
        RESOURCE[Resource Manager<br/>+ Async Loading]
        CACHE[Resource Cache<br/>+ LRU]
        THREAD[Thread Pool]
    end
    
    subgraph "Core Services"
        ERROR[Error Handler]
        PROFILER[Profiler]
        TIME[Time Manager]
        INPUT[Input Manager]
    end
    
    subgraph "Effects & UI"
        PARTICLE[Particle System<br/>+ GPU Simulation]
        UI[UI System<br/>+ Canvas]
        AUDIO[Audio System]
    end
    
    APP --> GAME
    GAME --> SCENE
    SCENE --> GO
    
    GO --> RENDERER
    GO --> COLLISION
    GO --> PARTICLE
    
    RENDERER --> BATCH
    RENDERER --> FRUSTUM
    RENDERER --> DEBUG
    
    COLLISION --> PHYSICS
    COLLISION --> RAYCAST
    
    RENDERER --> RESOURCE
    RESOURCE --> CACHE
    RESOURCE --> THREAD
    
    SCENE --> RESOURCE
    PARTICLE --> RENDERER
    UI --> RENDERER
    
    ERROR --> DEBUG
    PROFILER --> TIME
    
    style APP fill:#e1f5ff
    style SCENE fill:#fff4e1
    style GO fill:#e8f5e9
    style RENDERER fill:#fce4ec
    style RESOURCE fill:#f3e5f5
    style COLLISION fill:#e3f2fd
    style ERROR fill:#ffebee
```

## 12. メモリ管理ガイドライン

### 所有権の明確化

| ポインタ型 | 用途 | 使用場面 |
|-----------|------|---------|
| `unique_ptr<T>` | 単一所有権 | Component, Transform, 内部システム |
| `shared_ptr<T>` | 共有所有権 | GameObject, Resource, Model |
| `weak_ptr<T>` | 観察用参照 | Parent参照, キャッシュ, 循環参照回避 |
| `T*` (生ポインタ) | 一時参照 | 関数引数, 戻り値（所有権なし） |

### メモリ管理の実装例

```cpp
class GameObject {
private:
    // Transformは必須コンポーネントとして単一所有
    std::unique_ptr<Transform> m_Transform;
    
    // Componentは単一所有
    std::vector<std::unique_ptr<Component>> m_Components;
    
    // 親への参照は弱参照（循環参照防止）
    std::weak_ptr<GameObject> m_Parent;
    
    // 子は共有所有
    std::vector<std::shared_ptr<GameObject>> m_Children;

public:
    // Transformへのアクセス（所有権は渡さない）
    Transform* GetTransform() { 
        return m_Transform.get(); 
    }
    
    // コンポーネント追加（完全な所有権移譲）
    template<typename T, typename... Args>
    T* AddComponent(Args&&... args) {
        auto component = std::make_unique<T>(std::forward<Args>(args)...);
        T* ptr = component.get();
        m_Components.push_back(std::move(component));
        return ptr;
    }
};
```

## 13. エラーハンドリングパターン

### Result型の実装と使用例

```cpp
// Result型の定義
template<typename T>
class Result {
public:
    static Result<T> Success(T value) {
        Result<T> result;
        result.m_value = std::move(value);
        return result;
    }
    
    static Result<T> Error(std::string error) {
        Result<T> result;
        result.m_error = std::move(error);
        return result;
    }
    
    bool IsSuccess() const { 
        return m_value.has_value(); 
    }
    
    T& GetValue() { 
        return m_value.value(); 
    }
    
    const std::string& GetError() const { 
        return m_error; 
    }
    
private:
    std::optional<T> m_value;
    std::string m_error;
};

// 使用例：モデル読み込み
Result<std::shared_ptr<Model>> LoadModel(const std::string& path) {
    // ファイル存在チェック
    if (!std::filesystem::exists(path)) {
        return Result<std::shared_ptr<Model>>::Error(
            "File not found: " + path
        );
    }
    
    // モデル読み込み
    auto model = std::make_shared<Model>();
    if (!model->Load(path)) {
        return Result<std::shared_ptr<Model>>::Error(
            "Failed to parse model: " + path
        );
    }
    
    return Result<std::shared_ptr<Model>>::Success(model);
}

// 呼び出し側での処理
void InitializeScene() {
    auto result = LoadModel("assets/models/character.fbx");
    
    if (result.IsSuccess()) {
        auto model = result.GetValue();
        // モデルを使用
        m_Character->SetModel(model);
    } else {
        // エラー処理
        ErrorHandler::GetInstance().LogError(
            ErrorLevel::Warning, 
            result.GetError()
        );
        // フォールバック処理
        LoadDefaultModel();
    }
}
```

## 14. パフォーマンス最適化チェックリスト

### レンダリング最適化
- [x] フラスタムカリング実装
- [x] マテリアル/シェーダーごとのバッチング
- [x] インスタンシング対応
- [x] コマンドバッファによる描画最適化
- [ ] LOD（Level of Detail）システム
- [ ] オクルージョンカリング（オプション）

### メモリ最適化
- [x] オブジェクトプール（ParticlePool）
- [x] リソースキャッシュ with LRU
- [x] 弱参照による循環参照防止
- [x] 非同期リソースローディング
- [ ] メモリアロケータのカスタマイズ

### CPU最適化
- [x] 空間分割（QuadTree/OctTree）
- [x] マルチスレッド対応（ThreadPool）
- [x] コンポーネントのアップデート順序最適化
- [x] デルタタイム管理
- [ ] ジョブシステム実装

### GPU最適化
- [x] GPUパーティクルシミュレーション
- [x] Compute Shader活用
- [x] ConstantBuffer最適化
- [x] RenderState キャッシング
- [ ] GPU駆動レンダリング

## 15. 実装優先順位とロードマップ

### Phase 1: 基盤構築（1-2週間）
必須の基礎システムを実装

1. **Result型とエラーハンドリング** 
   - エラー処理の統一化
   - ログシステムの構築

2. **メモリ管理の統一**
   - smart pointerの導入
   - 所有権ルールの確立

3. **基本GameObject/Componentシステム**
   - Transform実装
   - Component基底クラス

4. **シンプルなレンダラー**
   - 基本的な描画機能
   - シェーダー管理

### Phase 2: コア機能（2-3週間）
ゲーム開発に必要な主要機能

5. **シーン管理システム**
   - シーン遷移
   - GameObject管理

6. **リソースマネージャー（同期版）**
   - テクスチャ/モデル読み込み
   - キャッシュシステム

7. **基本的なコリジョンシステム**
   - AABB/Sphere判定
   - コリジョンイベント

8. **入力管理**
   - キーボード/マウス
   - ゲームパッド対応

### Phase 3: 最適化（2-3週間）
パフォーマンス向上施策

9. **レンダリングコマンドバッファ**
   - 描画コマンドのキューイング
   - ソート最適化

10. **空間分割（QuadTree/OctTree）**
    - 効率的な衝突判定
    - カリング最適化

11. **非同期リソースローディング**
    - ThreadPool実装
    - プログレス表示

12. **バッチングシステム**
    - インスタンシング
    - マテリアルバッチング

### Phase 4: 高度な機能（3-4週間）
追加の便利機能

13. **GPUパーティクル**
    - Compute Shader実装
    - 大量パーティクル対応

14. **高度なUIシステム**
    - レイアウトシステム
    - イベントシステム

15. **プロファイラー**
    - パフォーマンス計測
    - ボトルネック特定

16. **デバッグレンダラー**
    - デバッグ表示
    - ギズモ描画

## 実装のベストプラクティス

### コーディング規約
```cpp
// ファイル名: PascalCase.h/.cpp
// クラス名: PascalCase
// 関数名: PascalCase
// 変数名: m_camelCase (メンバ変数)
//        camelCase (ローカル変数)
// 定数: UPPER_SNAKE_CASE
// 名前空間: lower_snake_case

// ヘッダーガード
#pragma once

// 前方宣言を活用
class GameObject;

// インクルードは最小限に
#include <memory>
#include <vector>
```

### デバッグマクロ
```cpp
#ifdef _DEBUG
    #define ASSERT(condition, message) \
        if (!(condition)) { \
            ErrorHandler::GetInstance().Assert(false, message); \
        }
    
    #define LOG(message) \
        ErrorHandler::GetInstance().LogInfo(message)
#else
    #define ASSERT(condition, message)
    #define LOG(message)
#endif
```

このフレームワーク設計により、堅牢で拡張性が高く、パフォーマンスも考慮されたゲームエンジンの構築が可能になります。
