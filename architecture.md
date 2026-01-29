# Swiss Ephemeris アーキテクチャドキュメント

## 概要

Swiss Ephemeris (SE) は、占星術・天文ソフトウェア向けの高精度天体暦計算ライブラリです。NASA JPL のデータに基づき、天文標準書と同等の精度で暦データを再現できます。

- **開発元**: Dieter Koch, Alois Treindl（Astrodienst AG, チューリッヒ）
- **バージョン**: 2.10.03（`sweph.h` の `SE_VERSION`）
- **ライセンス**: 二重ライセンス（AGPL v3 / Swiss Ephemeris Professional License）

---

## 技術スタック

| 項目 | 技術 |
|------|------|
| **言語** | C（ANSI C / C99） |
| **ビルド** | GNU Make |
| **コンパイラ** | `cc`（Linux: gcc/clang、macOS: Clang） |
| **依存ライブラリ** | `libm`（数学）、Linux のみ `libdl`（動的ロード） |
| **データ形式** | 独自圧縮暦（.se1）、JPL バイナリ（de200/de406/de431/de441.eph） |
| **テスト** | C + GNU m4 マクロ + Perl + Shell スクリプト |

---

## ディレクトリ構造

```
swisseph/
├── *.c, *.h              # コアライブラリ・アプリのソース
├── Makefile              # メインビルド（Linux/macOS）
├── bin/                  # ビルド済み実行ファイル（swetest, swevents）
├── ephe/                 # 暦データ（.se1, ep4/, sat/, astN/）
├── doc/                  # ドキュメント（PDF, HTML, 画像）
├── setest/               # テストスイート
├── contrib/              # サードパーティ寄贈（Android, Delphi, C#, VB 等）
└── windows/              # Windows 用（swewin, VB 宣言, プリビルド exe）
```

---

## コアライブラリ構成

### ビルド対象オブジェクト（libswe）

Makefile の `SWEOBJ` で定義されるコアオブジェクト:

| ファイル | 役割 |
|----------|------|
| **sweph.c** | メイン API。`swe_calc()` 等の公開関数、キャッシュ・ルーティング |
| **swephlib.c** | 内部計算（座標変換、章動、歳差、補正など） |
| **swecl.c** | 暦計算の中核（Ephemeris computations） |
| **swejpl.c** | JPL 暦の読み込み（DE200/403/405/406/431/441 対応） |
| **swedate.c** | 日付・ユリウス日（`swe_julday`, `swe_revjul`, `swe_deltat` 等） |
| **swehouse.c** | ハウス計算（`swe_houses`, `swe_house_pos` 等） |
| **swehel.c** | ヘリアカル・ライジング等（Victor Reijs 由来の翻訳） |
| **swemmoon.c** | 月の位置・運動 |
| **swemplan.c** | 惑星の位置・運動 |

### 主要ヘッダ

| ヘッダ | 用途 |
|--------|------|
| **swephexp.h** | 公開 API。アプリはこのファイルをインクルードしてリンク |
| **sweph.h** | 定数、惑星名、バージョン、内部用インデックス |
| **swephlib.h** | 内部用定義・定数（IAU 歳差等） |
| **sweodef.h** | 内部用型・定義 |
| **swejpl.h** | JPL 関連の内部定義 |
| **swehouse.h** | ハウス系定数 |
| **swedate.h** | 日付関連 |
| **swedll.h** | DLL エクスポート用（Windows） |

### その他のソース

| ファイル | 役割 |
|----------|------|
| **swetest.c** | コマンドラインテスト・デモ（`swetest`） |
| **swevents.c** | 天文イベント計算（`swevents`） |
| **swemini.c** | 最小限のテストプログラム（`swemini`） |
| **obama.c** | サンプル・デモプログラム |
| **sweephe4.c**, **swephgen4.c** | 暦ファイル生成用（メイン lib には含まれない） |

---

## ビルドシステム

### メイン Makefile

- **OS 検出**: `uname` で Linux / Darwin（macOS）を判定
- **Linux**: `libswe.so`（共有）、`libswe.a`（静的）、`swetests`（完全静的リンク版）
- **macOS**: `libswe.dylib`、`libswe.a`（静的リンクのみ、`swetests` は未対応）
- **ターゲット**: `all`, `swetest`, `swevents`, `swemini`, `obama`, `libswe.a`, `libswe.$(DYLIB_EXT)`, `test`, `clean`

### テスト（setest）

- **場所**: `setest/`
- **流れ**: `suite_*.c` と m4 マクロから `generated_tests.c` を生成 → `setest` をビルド
- **実行**: `make test`（ルートの Makefile から `cd setest && make && ./setest t`）
- **期待値**: `make test.exp` で `t.exp` を生成（master で期待値作成、devel で `make test`）

---

## 暦データ（ephe/）

- **パス**: デフォルトは `.:/users/ephe2/:/users/ephe/`（`swe_set_ephe_path()` で変更可）
- **主なファイル**:
  - **sepl_*.se1**, **seplm*.se1**: 惑星（短・長期間）
  - **semo_*.se1**, **semom*.se1**: 月
  - **seas_*.se1**, **seasm*.se1**: 小惑星
  - **ep4/**: 4 次補間用データ
  - **sat/**: 衛星（sepm*.se1）
- **小惑星**: `astN`（N = 小惑星番号/1000）に配置。短期（600年）・長期（6000年）の別あり
- **JPL**: de200.eph, de406.eph, de431.eph, de441.eph 等は別途ダウンロードし、`ephe` パス内に配置

---

## 公開 API の例（swephexp.h）

- **計算**: `swe_calc()`, `swe_calc_ut()`, `swe_fixstar()`, `swe_calc_pctr()` 等
- **日付**: `swe_julday()`, `swe_revjul()`, `swe_deltat()`
- **ハウス**: `swe_houses()`, `swe_houses_ex()`, `swe_house_pos()`
- **その他**: `swe_set_ephe_path()`, `swe_set_jpl_file()`, `swe_set_topo()`, `swe_set_sid_mode()`, `swe_close()`, `swe_version()`

---

## クロスプラットフォーム・連携

| 対象 | 場所・備考 |
|------|------------|
| **Windows** | `windows/`（swewin, リソース, VB 宣言）。Visual Studio 用は `contrib/projectPF_VS2017.zip` |
| **Android** | `contrib/android/jni/`（JNI + Android.mk）。`contrib/android/libs/` に .so |
| **Java** | 別リポジトリ（Thomas Mack）。contrib に旧 Java ソース zip |
| **Delphi** | `contrib/swissdelphi_*.zip` |
| **C# / .NET** | `contrib/rp_source_0_9.zip`（Radix Pro 等） |
| **VB/Excel** | `contrib/Sweph32_For_Excel_VBA_and_VB.zip`, `windows/vb/swedecl*.txt` |
| **PHP** | GitHub: php-sweph |
| **Perl** | GitHub: perl-sweph |

---

## ドキュメント

- **doc/**: `swephprg.pdf`（プログラマ向け）、`swisseph.pdf` / `swisseph.htm`、`secont_e.pdf` 等
- **オンライン**: https://www.astro.com/swisseph/

---

## ライセンス・注意

- 本プロジェクトは **AGPL-3.0** または **Swiss Ephemeris Professional License** のいずれかを選択して利用
- `contrib/` のコードは Astrodienst が内容を保証するものではなく、利用前に各自で確認が必要

---

*このドキュメントはプロジェクトのソースと Makefile、readme 等から生成した構成メモです。*
