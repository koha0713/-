# DirectX Game Framework - 機能チェックリスト & クイックリファレンス

## 📊 機能実装状況一覧

### コアシステム

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| メインループ | ✅ 完了 | 必須 | Application, Game |
| デルタタイム管理 | ✅ 完了 | 必須 | Timer クラス |
| シーン管理 | ✅ 完了 | 必須 | SceneManager |
| シーン遷移 | ✅ 完了 | 必須 | ChangeScene() |
| ウィンドウ管理 | ✅ 完了 | 必須 | Application |
| フルスクリーン切替 | ✅ 完了 | 中 | F11キー |
| ログシステム | ❌ 未実装 | 高 | **要実装** |
| 設定ファイル管理 | 🟡 骨組みのみ | 中 | DataManager |

### GameObjectシステム

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| GameObject基底 | ✅ 完了 | 必須 | GameObject クラス |
| Transform | ✅ 完了 | 必須 | Position, Rotation, Scale |
| Component基底 | ✅ 完了 | 必須 | Component クラス |
| AddComponent | ✅ 完了 | 必須 | テンプレート実装 |
| GetComponent | ✅ 完了 | 必須 | type_index マップ |
| RemoveComponent | ✅ 完了 | 必須 | - |
| タグシステム | ✅ 完了 | 中 | FindGameObjectWithTag() |
| アクティブ制御 | ✅ 完了 | 中 | SetActive() |
| 親子関係 | ❌ 未実装 | 中 | **要検討** |
| プレハブシステム | ❌ 未実装 | 低 | 時間があれば |

### グラフィックス

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| DirectX11初期化 | ✅ 完了 | 必須 | Renderer |
| 基本描画パイプライン | ✅ 完了 | 必須 | Renderer |
| シェーダー管理 | ✅ 完了 | 必須 | Shader クラス |
| テクスチャ読み込み | ✅ 完了 | 必須 | Texture クラス |
| OBJ/FBX読み込み | ✅ 完了 | 必須 | Assimp統合 |
| マテリアルシステム | ✅ 完了 | 必須 | Material クラス |
| カメラシステム | ✅ 完了 | 必須 | Camera クラス |
| ライティング | ✅ 完了 | 必須 | 平行光源のみ |
| MeshRenderer | ✅ 完了 | 必須 | MeshRendererComponent |
| 深度バッファ | ✅ 完了 | 必須 | Renderer |
| ブレンドモード | ✅ 完了 | 中 | BS_ALPHABLEND等 |
| 2D スプライト | 🟡 部分実装 | 高 | SimplePlane のみ |
| UI描画システム | ❌ 未実装 | 高 | **要実装** |
| テキスト描画 | ❌ 未実装 | 高 | **要実装** |
| パーティクル | ❌ 未実装 | 中 | **要実装** |
| スカイボックス | ❌ 未実装 | 低 | - |
| ポストエフェクト | ❌ 未実装 | 低 | - |
| シャドウマップ | ❌ 未実装 | 低 | - |

### 物理演算

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| PhysicsManager | ✅ 完了 | 必須 | 統合管理 |
| Collider基底 | ✅ 完了 | 必須 | Collider クラス |
| SphereCollider | ✅ 完了 | 必須 | 球形衝突判定 |
| AABBCollider | ✅ 完了 | 必須 | ボックス衝突判定 |
| Rigidbody | ✅ 完了 | 必須 | 物理演算 |
| 重力システム | ✅ 完了 | 必須 | PhysicsManager |
| 衝突イベント | ✅ 完了 | 必須 | Enter/Stay/Exit |
| レイヤーマスク | ✅ 完了 | 中 | 衝突フィルタリング |
| トリガー | ✅ 完了 | 中 | IsTrigger |
| CapsuleCollider | ❌ 未実装 | 中 | - |
| OBBCollider | ❌ 未実装 | 低 | - |
| レイキャスト | ❌ 未実装 | 中 | **要検討** |
| 物理マテリアル | ❌ 未実装 | 低 | 摩擦・反発係数 |
| ジョイント | ❌ 未実装 | 低 | - |

### リソース管理

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| ResourceManager | ✅ 完了 | 必須 | 統合管理 |
| テクスチャキャッシュ | ✅ 完了 | 必須 | shared_ptr管理 |
| メッシュキャッシュ | ✅ 完了 | 必須 | shared_ptr管理 |
| シェーダーキャッシュ | ✅ 完了 | 必須 | shared_ptr管理 |
| 非同期読み込み | ❌ 未実装 | 中 | ローディング画面用 |
| リソース監視 | ❌ 未実装 | 低 | ホットリロード |
| メモリプール | ❌ 未実装 | 低 | パフォーマンス向上 |

### オーディオ

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| SoundManager | ✅ 完了 | 必須 | XAudio2 |
| BGM再生 | ✅ 完了 | 必須 | PlayBGM() |
| SE再生 | ✅ 完了 | 必須 | PlaySE() |
| 音量調整 | ✅ 完了 | 必須 | SetVolume() |
| 停止・再開 | ✅ 完了 | 必須 | Stop(), Resume() |
| 3Dオーディオ | ❌ 未実装 | 中 | **要実装** |
| AudioSourceコンポーネント | ❌ 未実装 | 中 | **要実装** |
| ドップラー効果 | ❌ 未実装 | 低 | - |
| オーディオミキサー | ❌ 未実装 | 低 | - |

### 入力

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| IOManager | ✅ 完了 | 必須 | 統合管理 |
| キーボード入力 | ✅ 完了 | 必須 | Input クラス |
| ゲームパッド入力 | ✅ 完了 | 必須 | XInput |
| 入力バインド | ✅ 完了 | 中 | キーマッピング |
| トリガー/プレス/リリース | ✅ 完了 | 必須 | - |
| アナログスティック | ✅ 完了 | 必須 | GetLeftAnalogStick() |
| バイブレーション | ✅ 完了 | 低 | SetVibration() |
| マウス入力 | ❌ 未実装 | 中 | **要検討** |
| タッチ入力 | ❌ 未実装 | 低 | - |

### アニメーション

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| アニメーションシステム | ❌ 未実装 | 中 | **要検討** |
| スケルタルアニメーション | ❌ 未実装 | 中 | - |
| ブレンディング | ❌ 未実装 | 中 | - |
| IK (Inverse Kinematics) | ❌ 未実装 | 低 | - |
| アニメーションイベント | ❌ 未実装 | 低 | - |

### UI

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| UIManager | ❌ 未実装 | 高 | **要実装** |
| UIElement基底 | ❌ 未実装 | 高 | **要実装** |
| UIText | ❌ 未実装 | 高 | **要実装** |
| UIImage | ❌ 未実装 | 高 | **要実装** |
| UIButton | ❌ 未実装 | 高 | **要実装** |
| UISlider | ❌ 未実装 | 中 | - |
| UIPanel | ❌ 未実装 | 中 | - |
| レイアウトシステム | ❌ 未実装 | 中 | - |
| イベントシステム | ❌ 未実装 | 高 | onClick等 |

### デバッグ・開発支援

| 機能 | 実装状況 | 優先度 | 備考 |
|------|---------|--------|------|
| FPS表示 | ✅ 完了 | 必須 | Timer::GetFPS() |
| デルタタイム表示 | ✅ 完了 | 必須 | Timer::GetDeltaTime() |
| コンソール出力 | ✅ 完了 | 必須 | std::cout |
| ログシステム | ❌ 未実装 | 高 | **要実装** |
| デバッグ描画 | 🟡 骨組みのみ | 高 | **要実装** |
| コライダー可視化 | 🟡 骨組みのみ | 高 | DebugDraw() |
| プロファイラ | ❌ 未実装 | 中 | - |
| インスペクター | ❌ 未実装 | 低 | - |
| コマンドコンソール | ❌ 未実装 | 低 | - |

---

## 🎯 優先度別実装リスト

### 🔴 最優先（12月中旬まで）

#### 1. ログシステム
```cpp
Logger::Init("game.log");
Logger::Info("Game started");
Logger::Error("Failed to load texture");
```

**実装ファイル:**
- `Util/Logger.h`
- `Util/Logger.cpp`

**工数見積もり:** 0.5日

---

#### 2. UIシステム（基本）
```cpp
// UIManager
UIManager uiManager;
uiManager.Init();

// UIText
auto text = std::make_shared<UIText>();
text->SetText("Hello, World!");
text->SetPosition(Vector2(100, 100));
text->SetColor(Color(1, 1, 1, 1));
uiManager.AddUIElement(text);

// UIImage
auto image = std::make_shared<UIImage>();
image->SetTexture(texture);
image->SetPosition(Vector2(200, 200));
uiManager.AddUIElement(image);
```

**実装ファイル:**
- `Manager/UIManager.h/cpp`
- `GameObject/UIComponent/UIElement.h`
- `GameObject/UIComponent/UIText.h/cpp`
- `GameObject/UIComponent/UIImage.h/cpp`
- `GameObject/UIComponent/UIButton.h/cpp`

**工数見積もり:** 2-3日

---

#### 3. エラーハンドリング強化
```cpp
// カスタム例外
class GameException : public std::exception {
    // ...
};

// 使用例
try {
    auto mesh = ResourceManager::LoadMesh("model.obj");
} catch (const GameException& e) {
    Logger::Error(e.what());
}
```

**実装ファイル:**
- `Util/GameException.h`
- 既存のManager系クラスに追加

**工数見積もり:** 1日

---

### 🟡 中優先度（12月下旬まで）

#### 4. パーティクルシステム
```cpp
// パーティクル作成
auto* particles = gameObject->AddComponent<ParticleSystem>();

ParticleSystem::ParticleParameters params;
params.maxParticles = 100;
params.emissionRate = 20.0f;
params.lifetime = 2.0f;
params.startColor = Color(1, 0.5f, 0, 1);
params.endColor = Color(1, 0, 0, 0);

particles->SetParameters(params);
particles->Play();
```

**実装ファイル:**
- `GameObject/ParticleSystem.h/cpp`
- `shader/particleVS.hlsl`
- `shader/particlePS.hlsl`

**工数見積もり:** 2-3日

---

#### 5. デバッグ描画システム
```cpp
// 使用例
DebugDraw::DrawLine(start, end, Color(1, 0, 0, 1));
DebugDraw::DrawSphere(center, radius, Color(0, 1, 0, 1));
DebugDraw::DrawBox(center, size, Color(0, 0, 1, 1));
```

**実装ファイル:**
- `Util/DebugDraw.h/cpp`
- `shader/debugVS.hlsl`
- `shader/debugPS.hlsl`

**工数見積もり:** 1-2日

---

#### 6. 3Dオーディオシステム
```cpp
// AudioSourceコンポーネント
auto* audio = gameObject->AddComponent<AudioSourceComponent>();
audio->SetSound(SOUND_LABEL_EXPLOSION);
audio->Set3D(true);
audio->SetMinMaxDistance(5.0f, 50.0f);
audio->Play();
```

**実装ファイル:**
- `GameObject/AudioSourceComponent.h/cpp`
- `Manager/SoundManager.h` （拡張）

**工数見積もり:** 1-2日

---

### 🟢 低優先度（時間があれば）

#### 7. アニメーションシステム
#### 8. プレハブシステム
#### 9. マルチスレッド対応

---

## 📖 クイックリファレンス

### GameObject操作

```cpp
// GameObject作成
auto* player = scene->AddGameObject();
player->SetTag("Player");

// Transform操作
player->GetTransform().SetPosition(Vector3(0, 0, 0));
player->GetTransform().SetRotation(Vector3(0, 3.14f, 0));
player->GetTransform().SetScale(Vector3(1, 1, 1));

// コンポーネント追加
auto* renderer = player->AddComponent<MeshRendererComponent>();
auto* collider = player->AddComponent<SphereCollider>(1.0f);
auto* rigidbody = player->AddComponent<Rigidbody>();

// コンポーネント取得
auto* renderer = player->GetComponent<MeshRendererComponent>();

// コンポーネント削除
player->RemoveComponent<MeshRendererComponent>();

// GameObject検索
auto* player = scene->FindGameObjectWithTag("Player");
auto enemies = scene->FindGameObjectsWithTag("Enemy");
```

### リソース読み込み

```cpp
// テクスチャ
auto texture = M_RESOURCE.LoadTexture("asset/texture/player.png");

// メッシュ
auto mesh = M_RESOURCE.LoadMesh(
    "asset/model/character.obj",
    "asset/texture"
);

// シェーダー
auto shader = M_RESOURCE.LoadShader(
    "shader/litTextureVS.hlsl",
    "shader/litTexturePS.hlsl"
);
```

### 物理演算

```cpp
// Collider設定
auto* collider = gameObject->AddComponent<SphereCollider>(1.0f);
collider->SetTrigger(false);
collider->SetLayer(0);

// 衝突コールバック設定
collider->SetOnCollisionEnter([](const CollisionInfo& info) {
    Logger::Info("Collision Enter!");
});

// Rigidbody
auto* rb = gameObject->AddComponent<Rigidbody>();
rb->SetMass(1.0f);
rb->SetUseGravity(true);
rb->AddForce(Vector3(0, 10, 0));
rb->AddImpulse(Vector3(0, 5, 0));

// PhysicsManager設定
PHYSICS_MANAGER.SetGravity(Vector3(0, -9.8f, 0));
PHYSICS_MANAGER.SetLayerCollision(0, 1, true);
```

### 入力

```cpp
// キーボード
if (IO_MANAGER.GetKeyDown(TYPE_OK)) {
    // Enterキーが押された瞬間
}

if (IO_MANAGER.GetKeyPress(TYPE_OK)) {
    // Enterキーが押されている間
}

if (IO_MANAGER.GetKeyUp(TYPE_OK)) {
    // Enterキーが離された瞬間
}

// 直接キー指定
if (IO_MANAGER.GetKeyPressKeyBord(VK_W)) {
    // Wキーが押されている間
}
```

### サウンド

```cpp
// BGM再生
SOUND_MANAGER.PlayBGM(SOUND_LABEL_BGM000);

// SE再生
SOUND_MANAGER.PlaySE(SOUND_LABEL_SE000);

// 音量調整
SOUND_MANAGER.SetVolumeBGM(0.5f);
SOUND_MANAGER.SetVolumeSE(0.7f);

// 停止
SOUND_MANAGER.Stop(SOUND_LABEL_BGM000);
```

### シーン管理

```cpp
// シーン遷移
SCENE_MANAGER.ChangeScene(SCENE_GAME);
SCENE_MANAGER.ChangeScene(SCENE_TITLE);
SCENE_MANAGER.ChangeScene(SCENE_RESULT);
```

### カスタムコンポーネント作成

```cpp
// MyComponent.h
class MyComponent : public Component {
public:
    void Init() override {
        // 初期化処理
    }
    
    void Update() override {
        // 毎フレーム実行
        float deltaTime = Game::GetDeltaTime();
        
        // オーナーのTransform取得
        Transform& transform = m_pOwner->GetTransform();
        
        // 処理...
    }
    
    void Draw(Camera* camera) override {
        // 描画処理
    }
    
    void Uninit() override {
        // 終了処理
    }
};

// 使用例
auto* myComp = gameObject->AddComponent<MyComponent>();
```

---

## 🏗️ プロジェクト構造の推奨配置

```
YourGameProject/
│
├── src/                           # ソースコード
│   ├── Core/
│   ├── Manager/
│   ├── GameObject/
│   ├── Graphics/
│   ├── Scene/
│   ├── System/
│   └── Util/
│
├── include/                       # ヘッダーファイル
│
├── asset/                         # ゲームアセット
│   ├── model/                     # 3Dモデル
│   ├── texture/                   # テクスチャ
│   ├── shader/                    # シェーダー
│   ├── sound/
│   │   ├── BGM/
│   │   └── SE/
│   └── data/                      # 設定ファイル等
│
├── lib/                           # 外部ライブラリ
│   ├── DirectXTK/
│   └── assimp/
│
├── shader/                        # シェーダーファイル
│   ├── litTextureVS.hlsl
│   ├── litTexturePS.hlsl
│   ├── unlitVS.hlsl
│   └── unlitPS.hlsl
│
├── docs/                          # ドキュメント
│   ├── architecture.md
│   ├── api_reference.md
│   └── design_docs/
│
├── tests/                         # ユニットテスト
│
└── build/                         # ビルド成果物
```

---

## 📋 チェックリスト（提出前確認）

### コード品質
- [ ] すべてのクラスにドキュメントコメント
- [ ] マジックナンバーを定数化
- [ ] nullptrチェックの徹底
- [ ] メモリリークチェック
- [ ] 警告ゼロでコンパイル

### 機能
- [ ] ゲームが起動する
- [ ] シーン遷移が動作する
- [ ] 基本的な操作ができる
- [ ] 音が鳴る
- [ ] UI が表示される
- [ ] 当たり判定が動作する

### パフォーマンス
- [ ] 60FPS維持
- [ ] ローディング時間が妥当
- [ ] メモリ使用量が妥当

### ドキュメント
- [ ] README.mdの作成
- [ ] ビルド手順の記載
- [ ] 操作説明の記載
- [ ] 設計ドキュメント

### テスト
- [ ] すべてのシーンで動作確認
- [ ] エラー時の挙動確認
- [ ] 長時間実行でのメモリリークチェック

---

## 🚀 次のステップ

### 今すぐやるべきこと
1. ログシステムの実装
2. UIシステムの基本実装
3. エラーハンドリングの強化

### 今週中にやるべきこと
4. デバッグ描画システム
5. ゲームコンテンツの設計

### 来週やるべきこと
6. パーティクルシステム
7. ゲームコンテンツの実装開始

### 最終週にやるべきこと
8. バグ修正
9. 最終調整
10. ドキュメント完成

頑張ってください！💪🎮
