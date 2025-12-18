# Kinetic Link Physics

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Matter.js](https://img.shields.io/badge/Physics-Matter.js-00d1b2.svg)](https://brm.io/matter-js/)
[![Tailwind CSS](https://img.shields.io/badge/UI-Tailwind_CSS-38B2AC.svg)](https://tailwindcss.com/)

> **物理制約の動的再構成による、プロトタイプ型アクションパズル・フレームワーク。**

**[🕹 Live Demo (GitHub Pages)](https://funmatu.github.io/kinetic-link-physics/)**

---

## 1. プロジェクト・コンセプト
本プロジェクトは、2D物理演算ライブラリ「Matter.js」を用いた、**「質量連結」と「制御転送」** をテーマとする技術デモンストレーション、およびパズルゲームのプロトタイプです。

従来の固定されたキャラクター操作ではなく、動的に自身の構成パーツ（剛体）を増減させ、重心や物理的制約を変化させながら環境を攻略するメカニズムの構築を目的としています。

---

## 2. コア・アーキテクチャ

### 2.1 物理演算の統合
本システムは、HTML5 Canvas上でリアルタイム物理シミュレーションを行います。各オブジェクトは「Rigid Body（剛体）」として定義され、以下の物理パラメータが動的に適用されます。

* **Friction（摩擦）:** 0.1
* **Restitution（反発係数）:** 0.1
* **Mass（質量）:** ブロックのサイズに応じた自動計算（$m = \rho \times V$）

### 2.2 動的制約システム (Dynamic Constraint System)
プレイヤーが特定の範囲（`SHIFT_RANGE: 200px`）内にあるオブジェクトに干渉した際、`Matter.Constraint` を生成して物理的な結合を行います。



#### 連結のアルゴリズム
1.  **距離計算:** プレイヤーを構成する全ブロックからターゲットまでの最短距離を算出。
2.  **制約の生成:** 最も近いブロック（`bodyA`）とターゲット（`bodyB`）を結合。
    * `stiffness（剛性）`: 0.8（僅かな遊びを持たせることで物理的な挙動の面白さを演出）
    * `length（長さ）`: 接触時の距離を維持

---

## 3. 技術的特徴とロジック解説

### 3.1 制御（意識）の転送メカニズム
本プロジェクトのユニークな点は、プレイヤーの操作対象が「単一のエンティティ」ではないことです。
* **Transfer Control:** `transferControl(target)` 関数が実行されると、既存の全制約が破棄され、操作権（入力ベクトルの適用対象）が新しい剛体へと瞬時に移行します。これにより「肉体を乗り換えて進む」というパズル要素を実現しています。

### 3.2 接地判定とジャンプ・ロジック
物理エンジン上で不自然な空中ジャンプを防ぐため、プレイヤー構成パーツの `velocity.y`（垂直方向速度）を監視しています。
$$|v_y| < 0.2$$
この条件を満たすパーツが一つでも存在する場合のみ、上方への力（`applyForce`）の適用を許可しています。

### 3.3 衝突フィルタリングとハザード検知
`Events.on(engine, 'collisionStart', ...)` を利用し、ラベルベースの衝突判定を行っています。
* **Goal判定:** プレイヤーパーツが `goal` ラベルの物体と接触。
* **Hazard判定:** プレイヤーパーツが `hazard` ラベル（溶岩・トゲ等）に接触した際、物理計算ループを一時停止せずに、次のティックで非同期的に `loadLevel` を呼び出すことで、エンジンのクラッシュを回避しています。

---

## 4. モジュール構成

### クラス構造（擬似）
* **Engine & Runner:** Matter.jsのメインループ管理。
* **Level Loader:** `LEVELS` 配列からオブジェクトを抽出し、`World.add` で物理空間へ展開。
* **Input Handler:** キーボード入力（WASD/矢印）を物理的な力（Force）に変換。
* **Visualizer:** Matter.jsのデフォルト描画に加え、Canvas APIを用いた「目の描画（プレイヤー状態の視覚化）」をオーバーレイ。

---

## 5. ステージ設計定義 (JSON)
レベルデザインは、拡張性を考慮して以下のような構造で定義されています。

```javascript
{
    x: 100,      // 初期X座標
    y: 500,      // 初期Y座標
    w: 40,       // 幅（オプション）
    h: 40,       // 高さ（オプション）
    type: 'static' // 物理属性 (player, stickable, goal, hazard, static)
}

```

---

## 6. 操作方法 (Technical Interaction)

| 操作 | 物理的アクション |
| --- | --- |
| **W / Space** | 構成ブロック全方位への垂直力適用 (`Body.applyForce`) |
| **A / D** | 水平方向への持続的な加速度適用 |
| **Mouse Click** | レイキャストおよび距離計算に基づく制約の生成・破棄 |
| **R** | シミュレーションの破棄および再インスタンス化 |

---

## 7. 開発環境のセットアップ

1. **リポジトリのクローン**
```bash
git clone https://github.com/Funmatu/kinetic-link-physics.git

```


2. **依存関係**
* CDN経由で `Matter.js` および `Tailwind CSS` を読み込んでいるため、ビルドプロセスは不要です。


3. **ローカル実行**
* `index.html` を任意のブラウザ（Chrome/Edge/Firefox推奨）で開くだけで動作します。



---

## 8. ライセンス

本プロジェクトは **MIT License** の下で公開されています。商用・非商用を問わず、自由な改変・再配布が可能です。
