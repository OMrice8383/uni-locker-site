# P-02 Web用モデル（検証版）

`jersey.glb` は丸首10パーツ＋ハンガーのみ。シャツ30,380三角形、ハンガー5,364三角形、846,492 bytes。テクスチャ・カメラ・ライト・ポロは含まない。元FBX／ZPRJ／blendはGit対象外。

- Y上、正面+Z、着用者左+X。ハンガー上端を原点、シャツの高さを1に均一正規化。
- D-33の10島・境界・残存頂点UVを保持。裏面・厚みを除いた両面描画（裏も同じ柄）。glTFのVは画像の下向きなので、Three.jsで外部画像を貼る際は `flipY=false`、`colorSpace=SRGBColorSpace`。
- Nodeの `extras.role` は `shirt` / `hanger`。ハンガーのNode名は `Object.003`、Mesh名は元データ由来の `Obj_Avatar_Shape.002`。Mesh名とNode名を混同しない。
- 反転を起こす頂点と元の1-ringを固定してMeshoptimizer 1.3.0で削減。元UVと、実際のfloat32 glTF UVの両方で反転0枚。予算3万から6回の試算、採用30,380（承認上限4.5万内）。

## テクスチャと確認画面

`fp-home-2048.webp`／`fp-home-4096.webp` はClaudeの `assets-src/kit/fp-home/atlas-v0.png` をWeb用に変換したlossless WebP。2048版だけLANCZOSで縮小。`*-15.webp` はRYOTO・15の事前合成比較用。書体は仮のArial Bold、配置も試作。これを完成キットとして使わない。

```powershell
npm run dev -- --port 5187
# http://127.0.0.1:5187/?preview=3d
npm run check
```

P-02の確認画面は開発時限定。本番の `?preview=3d` は通常Appを表示し、比較用の確認コードは本番JSに含まれない。P-03以降の通常AppはThree/R3Fを別chunkから遅延読み込みする。解像度／事前合成／Canvas合成、前後左右、ドラッグ・矢印キーを切り替えられる。

計測値は `validation.json`、詳しい手順・結果は `docs/PI_IMPLEMENTATION_HANDOFF.md` §12と `docs/TESTING.md` §13。GPUメモリはRGBA8＋mipの概算、Windows Chromeの390×844は実機スマホではない。

通常Appはラック1024px、選択中2048pxの事前合成を使う。4096はP-02のロゴ拡大確認用。全18人分をGPUへ同時ロードしない。実物の字形・背面・袖パッチの照合と実機検証は残作業。

## P-03の選手別テクスチャ

`players/<id>-1024.webp`／`<id>-2048.webp` は正式なv0字形を使う合成画像。`assets-src/kit/numbers/` のSVG・layout.json・Arimo-Bold.ttfを読み、背面番号の下にネーム、前面の右胸に小さい番号を置く。`compose.mjs` のSVGと完全一致するテストがある。v0字形のうち2・4・6は推定（原本README参照）。

```powershell
npm install --prefix .local/tools @resvg/resvg-js  # 生成専用。アプリの依存ではない
# Python 3.11 + Pillowを使う
node src/scene/compose-player-textures.mjs ryoto-kai 15 RYOTO
node src/scene/compose-player-textures.mjs keita-kumokiri 19 K.KEITA
```

4枚のlossless WebPを生成し、選手別JSONに番号・確認済みネーム・公式販売案内のURL・元アトラスと出力のSHAを残す。2048／1024への縮小はLANCZOS。P-05の確認済みネームをCLIに渡すが、今回許可されたroster編集はjerseyTexturesだけなのでshirtNameは変更しない。

通常表示: `http://127.0.0.1:5187/`。開発時の18着確認: `?preview=locker`（実在の2人を複製した負荷確認で、18人分の名簿ではない）。GLBのgeometryを共有し、中心±3着の1024と選択中1着の2048だけをGPUへ載せる。範囲外のテクスチャとmaterialは解放する。P-03の計測はvalidation.jsonのlocker_scene_p03、手順はTESTING §15。

P-04では`useScene3DSettings()`に接続。開発時の通常画面`?tune=1`で、3Dタブの全項目と引き出し/回転/揺れのReplayが反映される。2D表示中は3Dがアンマウントされ、3Dタブは未接続になる。保存値・元GLB/UVは変更していない。再検証の数値・読み込み世代の旧エラー回帰はvalidation.jsonのlocker_scene_p04とTESTING §16。P-03/P-04は別コミット。2026-10-10にc641125へrebaseし、今回のユーザー依頼で専用ブランチへpushして渡す。main統合はClaude確認後、公開は未実施。

### 縮小版番号（2026-10-10、c641125）

4枚の現行画像は更新済みlayout/digit SVGを読み直して生成した。新しいletterSpacingも接続済み。Arimo BoldはSVGへ実TTFを埋込み、生成器にもそのファイルだけを指定（システムの代替書体は使わない）。`compose.mjs 10 NAME out.svg`を含む3例のSVGと、縮小後の配信用4枚の画素が参照と一致する。

比較: [背面](../../docs/images/p03-followup/number-10-comparison-back.png)／[前面](../../docs/images/p03-followup/number-10-comparison-front.png)（2026-10-10に配信フォルダからdocsへ移動）。左は参照SVG4096、右は合成→2048 WebP。実GLB・同じカメラ/照明で描画した10/NAMEの比較で、選手登録はしていない。Arimo Bold/weight700と書体読込を確認済み。詳細はTESTING §17・validation.jsonのnumber_layout_update。旧locker_scene_p03/p04の速度/負荷値は再計測していない。

### P-03 follow-up（main d41e607起点）

番号比較PNGはdocsへ移し、ビルド配信物に含めない。スマホの初期camera.mobileDistanceだけ5.2→3.04に同期（src/tuner/settings.tsとtuning.json）。選択時はシャツの実GLB中心に注視点を合わせ、ステージ幅60〜61%に表示。PCは元のカメラ・姿勢、照明の保存値も不変。照明はTunerの3案を撮影しただけで未採用。画像・検証はTESTING §18を参照。

素材単体で再配布しない。今回の範囲はprivate GitHubへの記録とローカル確認で、公開・対外配布は未実施。再生成用のローカル工程は `.local/models/simplify-locked.mjs` → `export-locked.py` → `web-textures.py`（Blenderは `--disable-autoexec --python-exit-code 1` を付ける）。元モデルを保持しているPi作業環境で実行する。
