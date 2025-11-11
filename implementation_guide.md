# 📘 DirectX11 フレームワーク完成ガイド

## 🎯 目標

**12月末までに実用的なゲーム開発フレームワークを完成させる**

---

## 📅 実装スケジュール (7週間)

### Week 1-2: 基盤整備 ⭐ 最優先

```
目標: ResourceManager と GameObject システムの基礎を完成させる
```

#### Day 1-3: ResourceManager 実装

**ファイル:**
- `Manager/ResourceManager.h`
- `Manager/ResourceManager.cpp`

**実装内容:**
```cpp
✅ LoadTexture() - キャッシュ機能
✅ LoadMesh() - キャッシュ機能  
✅ LoadShader() - キャッシュ機能
✅ ClearAll() - リソース解放
✅ PrintCacheInfo() - デバッグ情報
```

**テスト方法:**
```cpp
// 同じファイルを2回読み込んでキャッシュ動作を確認
auto tex1 = M_RESOURCE.LoadTexture("test.png");
auto tex2 = M_RESOURCE.LoadTexture("test.png"); // キャッシュヒット!
assert(tex1 == tex2); // 同じポインタ

M_RESOURCE.PrintCacheInfo(); // "Textures: 1" と表示されるはず
```

#### Day 4-7: GameObject と Component 基礎

**ファイル:**
- `GameObject/GameObject.h` (既存の Object.h を置き換え)
- `GameObject/Component.h`
- `GameObject/Transform.h`

**実装内容:**
```cpp
✅ Transform クラス
   - Position, Rotation, Scale
   - GetWorldMatrix()

✅ Component 基底クラス
   - Init(), Update(), Draw()
   - SetOwner(), GetOwner()

✅ GameObject クラス  
   - AddComponent<T>()
   - GetComponent<T>()
   - Update(), Draw()
```

**テスト方法:**
```cpp
// TestCube シーンで動作確認
auto obj = std::make_unique<GameObject>();
obj->GetTransform().SetPosition(Vector3(0, 0, 0));
obj->Update();
obj->Draw(camera);
```

#### Day 8-10: MeshRendererComponent 実装

**ファイル:**
- `GameObject/MeshRendererComponent.h`

**実装内容:**
```cpp
✅ SetMesh() - メッシュ設定
✅ SetShader() - シェーダー設定
✅ Draw() - サブセット描画
✅ マテリアル自動生成
```

**テスト方法:**
```cpp
auto obj = std::make_unique<GameObject>();
auto renderer = obj->AddComponent<MeshRendererComponent>();
renderer->SetMesh(M_RESOURCE.LoadMesh("model/cube.obj"));
renderer->SetShader(M_RESOURCE.LoadShader("shader/lit.vs", "shader/lit.ps"));
obj->Draw(camera); // キューブが描画される
```

#### Day 11-14: SceneBase 改良

**ファイル:**
- `Scene/SceneBase.h`
- `Scene/SceneBase.cpp`

**実装内容:**
```cpp
✅ AddGameObject<T>() - GameObject追加
✅ UpdateObjectList() - 全更新
✅ DrawObjectList() - 全描画  
✅ DeleteObjectList() - 全削除
```

**テスト方法:**
```cpp
// SceneTitle で複数オブジェクトを配置
void SceneTitle::Init() {
    auto obj1 = AddGameObject<GameObject>();
    auto obj2 = AddGameObject<GameObject>();
    auto obj3 = AddGameObject<GameObject>();
    // ... 動作確認
}
```

---

### Week 3-4: コンポーネント拡張

```
目標: 実用的なコンポーネントを追加してゲーム制作の準備
```

#### Week 3: 基本コンポーネント

**実装するコンポーネント:**

1. **CameraComponent** (オプション)
```cpp
class CameraComponent : public Component {
    // GameObject にカメラ機能を追加
    void Update() override;
};
```

2. **LightComponent** (オプション)
```cpp
class LightComponent : public Component {
    // GameObject にライト機能を追加
};
```

3. **ScriptComponent** (ユーザー定義)
```cpp
class PlayerController : public Component {
    void Update() override {
        // プレイヤーの操作
        if (IO_MANAGER.GetKeyPress(VK_W)) {
            m_pOwner->GetTransform().Translate(Vector3(0, 0, 0.5f));
        }
    }
};
```

#### Week 4: ゲームオブジェクト例

**ファイル:**
- `GameObject/PlayerObject.h` (オプション)
- `GameObject/EnemyObject.h` (オプション)

**実装例:**
```cpp
// PlayerObject.h (GameObject を継承してもOK)
class PlayerObject : public GameObject {
public:
    void Init() {
        // MeshRenderer 追加
        auto renderer = AddComponent<MeshRendererComponent>();
        renderer->SetMesh(M_RESOURCE.LoadMesh("model/player.obj"));
        
        // コントローラ追加
        AddComponent<PlayerController>();
    }
};
```

---

### Week 5-7: ゲーム制作 🎮

```
目標: 実際にゲームを作ってフレームワークを検証
```

#### SceneGame 実装

**ファイル:**
- `Scene/SceneGame.h`
- `Scene/SceneGame.cpp`

**実装内容:**
```cpp
void SceneGame::Init() {
    // プレイヤー配置
    auto player = AddGameObject<GameObject>();
    player->GetTransform().SetPosition(Vector3(0, 0, 0));
    auto playerRenderer = player->AddComponent<MeshRendererComponent>();
    playerRenderer->SetMesh(M_RESOURCE.LoadMesh("model/player.obj"));
    player->AddComponent<PlayerController>();
    
    // 敵配置 (10体)
    for (int i = 0; i < 10; i++) {
        auto enemy = AddGameObject<GameObject>();
        // ... 配置処理
    }
    
    // ステージ配置
    auto stage = AddGameObject<GameObject>();
    // ... 配置処理
}
```

**ゲーム要素:**
- プレイヤー操作 (WASD移動)
- 敵AI (簡易的でOK)
- 衝突判定 (簡易的でOK)
- ゲームオーバー/クリア条件

---

## 📂 ファイル構成 (最終形)

```
project/
├── Manager/
│   ├── ResourceManager.h       ← 新規実装 ⭐
│   ├── ResourceManager.cpp     ← 新規実装 ⭐
│   ├── SceneManager.h          ← 既存
│   ├── DataManager.h           ← 既存
│   ├── SoundManager.h          ← 既存
│   └── IOManager.h             ← 既存
│
├── GameObject/
│   ├── GameObject.h            ← 新規実装 ⭐ (Object.h を置き換え)
│   ├── Component.h             ← 新規実装 ⭐
│   ├── Transform.h             ← 新規実装 ⭐
│   ├── MeshRendererComponent.h ← 新規実装 ⭐
│   ├── PlayerController.h      ← 新規実装 (例)
│   └── ...                     ← 必要に応じて追加
│
├── Scene/
│   ├── SceneBase.h             ← 改良 ⭐
│   ├── SceneBase.cpp           ← 改良 ⭐
│   ├── SceneTitle.h            ← 改良
│   ├── SceneGame.h             ← 新規実装 ⭐
│   └── Scene.h                 ← SceneGame 追加
│
├── Graphics/
│   ├── Renderer.h              ← 既存
│   ├── Shader.h                ← 既存
│   ├── Texture.h               ← 既存
│   ├── VertexBuffer.h          ← 既存
│   └── ...
│
├── Core/
│   ├── Application.h           ← 既存
│   └── Game.h                  ← 既存 (ResourceManager.Init() 追加)
│
└── ...
```

---

## 🔧 実装時の注意点

### 1. Game.cpp の初期化順序

```cpp
void Game::Init() {
    Renderer::Init();
    
    // ResourceManager を最初に初期化 ⭐
    M_RESOURCE.Init();
    
    SCENE_MANAGER.Init();
    DATA_MANAGER.Init();
    SOUND_MANAGER.Init();
    IO_MANAGER.Init();
    
    m_Camera.Init();
}

void Game::Uninit() {
    m_Camera.Uninit();
    
    SCENE_MANAGER.UnInit();
    
    // ResourceManager を最後に解放 ⭐
    M_RESOURCE.UnInit();
    
    Renderer::Uninit();
    
    SingletonFinalizer::finalize();
}
```

### 2. Scene.h への SceneGame 追加

```cpp
// Scene.h
#include "SceneTitle.h"
#include "SceneGame.h"    // 追加 ⭐
#include "TestCube.h"

enum SCENE {
    SCENE_TITLE,
    SCENE_GAME,           // 追加 ⭐
    SCENE_CUBE,
    SCENE_NUM,
};

class Scene {
public:
    Scene() {
        m_sceneTable[SCENE_TITLE] = std::make_unique<SceneTitle>();
        m_sceneTable[SCENE_GAME] = std::make_unique<SceneGame>();  // 追加 ⭐
        m_sceneTable[SCENE_CUBE] = std::make_unique<TestCube>();
    }
};
```

### 3. TestCube の GameObject 化 (オプション)

```cpp
// TestCube を GameObject として再実装
class CubeObject : public GameObject {
public:
    void Init() {
        GetTransform().SetPosition(Vector3(0, 0, 0));
        
        auto renderer = AddComponent<MeshRendererComponent>();
        // ... メッシュ設定
    }
};

// SceneGame で使用
void SceneGame::Init() {
    auto cube = AddGameObject<CubeObject>();
    cube->Init();
}
```

---

## ✅ チェックリスト

### Phase 1: 基盤整備 (Week 1-2)

- [ ] ResourceManager 実装
  - [ ] LoadTexture() 実装
  - [ ] LoadMesh() 実装
  - [ ] LoadShader() 実装
  - [ ] キャッシュ動作確認
  
- [ ] GameObject 実装
  - [ ] Transform クラス
  - [ ] Component 基底クラス
  - [ ] AddComponent<T>() 動作確認
  - [ ] GetComponent<T>() 動作確認
  
- [ ] MeshRendererComponent 実装
  - [ ] SetMesh() 実装
  - [ ] SetShader() 実装
  - [ ] Draw() 実装
  - [ ] 描画確認
  
- [ ] SceneBase 改良
  - [ ] AddGameObject<T>() 実装
  - [ ] UpdateObjectList() 実装
  - [ ] DrawObjectList() 実装
  - [ ] 動作確認

### Phase 2: コンポーネント拡張 (Week 3-4)

- [ ] PlayerController 実装
- [ ] EnemyAI 実装 (簡易)
- [ ] 基本ゲームオブジェクト作成
- [ ] SceneGame 基本実装

### Phase 3: ゲーム制作 (Week 5-7)

- [ ] プレイヤー操作完成
- [ ] 敵AI完成
- [ ] 衝突判定実装
- [ ] ゲームロジック完成
- [ ] タイトル/リザルトシーン完成

---

## 🎓 学習リソース

### 推奨ドキュメント

1. **Unity のドキュメント**
   - Component システムの理解
   - GameObject の使い方
   
2. **Game Programming Patterns**
   - Component パターン
   - Resource Manager パターン

3. **DirectX 11 チュートリアル**
   - レンダリング最適化
   - リソース管理

---

## 🐛 トラブルシューティング

### よくある問題

#### 1. リソースが見つからない

```
[ResourceManager] Failed to load texture: model/player.png
```

**解決策:**
- ファイルパスを確認
- 作業ディレクトリを確認
- ファイルが実際に存在するか確認

#### 2. コンポーネントが取得できない

```cpp
auto renderer = obj->GetComponent<MeshRendererComponent>();
assert(renderer); // nullptr!
```

**解決策:**
- AddComponent() が呼ばれているか確認
- 型が一致しているか確認 (`<MeshRendererComponent>`)

#### 3. 描画されない

**確認項目:**
- カメラが正しく設定されているか
- オブジェクトが有効 (IsActive()) か
- メッシュとシェーダーがセットされているか
- Transform の位置が適切か

---

## 🎉 完成後の効果

このフレームワークを完成させると:

✅ **開発効率が5倍以上向上**
   - オブジェクトを追加するだけ
   - リソース管理を気にしなくてOK
   
✅ **メモリ効率が大幅に改善**
   - 同じモデルを100個配置してもメモリは1個分
   
✅ **チーム開発がスムーズ**
   - 各メンバーが独立して作業可能
   - コンフリクトが減少
   
✅ **保守性が向上**
   - バグ修正が容易
   - 機能追加が簡単

---

## 📞 サポート

実装中に困ったら:
1. このドキュメントを再確認
2. 提供したサンプルコードを参照
3. デバッグ情報 (PrintCacheInfo など) を活用
4. 一つずつ段階的に実装

**頑張ってください! 12月末までに完成させましょう! 🚀**
