# Windows ディレクトリ アーキテクチャ

## 概要

`windows/` は Swiss Ephemeris の Windows 向け配布物とサンプルを格納するディレクトリです。DLL・実行ファイルのプリビルド、GUI サンプル（swewin）、Visual Basic 用宣言、および Windows 用パッケージ（sweph.zip）の内容説明を含みます。

---

## ディレクトリ構造

```
windows/
├── readme.md           # 本ディレクトリの説明
├── architecture.md     # 本ドキュメント（Windows 配下の構成）
├── sweph.zip           # Windows 用パッケージ（DLL・ソース・VB・doc を含む）
├── swephzip.txt        # sweph.zip 内ファイル一覧
├── programs/           # プリビルド済みコマンドライン・GUI サンプル
├── swewin/             # GUI サンプル「SwissEph test」のソース（32/64bit）
└── vb/                 # Visual Basic 用 API 宣言（32/64bit）
```

---

## 各要素の説明

### readme.md

- 本フォルダの目的と `sweph.zip` の概要
- `swephzip.txt` がパッケージ内容の一覧であることの説明
- `programs/`（コマンドライン用バイナリ）、`swewin/`（GUI サンプル）、`vb/`（VB 宣言）の役割

### sweph.zip と swephzip.txt

- **sweph.zip**: Windows 用にまとめたパッケージ。解凍すると `sweph/` 以下に次のような構成になる（swephzip.txt の内容に基づく）。

| パス | 内容 |
|------|------|
| **sweph/bin/** | 32/64bit DLL（swedll32.dll, swedll64.dll）、インポートライブラリ（.lib）、サンプル exe（swetest, swete, swewin 等） |
| **sweph/src/** | C ソース（sweph.c, swephlib.c, swejpl.c, swetest.c 等）、ヘッダ、暦用テキスト（seasnam.txt, sefstars.txt 等） |
| **sweph/src/projects/** | Visual Studio 用 vcxproj / sln（swedll32, swedll64, swetest, swete, swewin32, swewin64 等） |
| **sweph/src/swewin/** | GUI サンプル用ソース（swewin.c, swewin.rc, resource.h 等） |
| **sweph/vb/** | VB 用宣言（swedecl.txt, swedecl64.txt）、.lib、サンプル（orbit.xls, sweph_vb7_64.bas）、Sweph32_For_Excel_VBA_and_VB.zip 等 |
| **sweph/doc/** | ドキュメント（swisseph.htm/pdf, swephprg.htm/pdf）、画像（sweph.gif 等） |

- **swephzip.txt**: 上記 zip のアーカイブ一覧（ファイル名・サイズ・日付）。内容確認用。

※ zip 内のソースはリリース時点のスナップショットであり、メイン git リポジトリより古い場合があります。

---

## programs/（プリビルド実行ファイル）

Windows 用にビルドされたサンプル・ユーティリティの exe を格納します。

| ファイル | 種別 | 説明 |
|----------|------|------|
| **swetest.exe** | 32bit | コマンドライン暦テスト（メインリポの swetest に相当） |
| **swetest64.exe** | 64bit | 上記の 64bit 版 |
| **swete32.exe** | 32bit | コマンドライン用サンプル（swete） |
| **swete64.exe** | 64bit | 上記の 64bit 版 |
| **swevents64.exe** | 64bit | 天文イベント計算（swevents に相当） |
| **swewin32.exe** | 32bit | GUI サンプル「Calculate Planets and Houses」（swewin のビルド結果） |
| **ceres.exe** | - | 小惑星位置計算のスタンドアロンサンプル（SE 本体のサポート対象外） |
| **ceres.readme** | - | ceres.exe の利用・免責の説明 |

実行時は `swedll32.dll` / `swedll64.dll` を同じディレクトリまたはシステムパスに置く必要があります（zip 内の bin に同梱）。

---

## swewin/（GUI サンプル）

Swiss Ephemeris の Windows 用 GUI デモ「SwissEph test」のソースです。日付・時刻・経緯度・ハウス系を入力し、惑星・ハウス・恒星などを計算して表示します。

| ファイル | 役割 |
|----------|------|
| **swewin.c** | メイン実装（ダイアログ、swe_* 呼び出し、Win32 API）。32bit ビルドで使用。 |
| **swewin64.c** | 64bit 用の同一プログラム（ポインタ・ハンドルを 64bit 対応にした版）。 |
| **swewin.h** | 32bit 用ヘッダ（Win16/Win32 互換マクロ、`__export` 等）。 |
| **swewin64.h** | 64bit 用ヘッダ（`GetWindowLongPtr` 等）。 |
| **swewin.rc** | リソーススクリプト（ダイアログ「Calculate Planets and Houses」、コントロール ID、メニュー等）。 |
| **resource.h** | リソース用 ID 定義（EDIT_DAY, COMBO_HSYS, PB_DOIT 等）。 |

- ビルドはメインリポの `contrib/projectPF_VS2017.zip` の Visual Studio プロジェクトを使うか、sweph.zip 内の `sweph/src/projects/swewin32.vcxproj` / `swewin64.vcxproj` を使用。
- readme では「compiled (and outdated)」とされており、最新の SE API や UI に更新されていない可能性があります。

---

## vb/（Visual Basic 用）

Visual Basic（VBA/Excel 含む）から Swiss Ephemeris の DLL を呼ぶための宣言と、64bit 用の注意書きです。

| ファイル | 用途 |
|----------|------|
| **swedecl.txt** | 32bit VB 用 API 宣言。`swedll32.dll` の `Declare` 文（`swe_calc`, `swe_julday`, `swe_houses` 等）。 |
| **swedecl64.txt** | 64bit VB 用 API 宣言。`swedll64.dll` を `Declare PtrSafe` で宣言。文字列・ポインタの扱いに関するコメントあり。 |

- DLL は実行ファイルと同じディレクトリか、システムの検索パスに配置。
- より充実した Excel/VBA サンプルは `contrib/Sweph32_For_Excel_VBA_and_VB.zip` や sweph.zip 内の `sweph/vb/` を参照。

---

## ビルド・開発時の注意

- **DLL の再ビルド**: メインリポの C ソースを変更した場合、Windows 用 DLL は Visual Studio 等で別途ビルドが必要。sweph.zip 内の `sweph/src/projects/sweph.sln` および各 vcxproj を参照。
- **プロジェクトファイル**: 最新の VS 用は `contrib/projectPF_VS2017.zip` を展開して、sweph.zip の `src` と組み合わせて使用する想定（contrib/readme 参照）。
- **swewin**: リポジトリ直下の `windows/swewin/` はソースのみ。実際の exe は `programs/` または sweph.zip の bin に含まれる。

---

## 関連リンク・ファイル

- メインリポジトリ: https://github.com/aloistr/swisseph  
- Windows 配布: https://github.com/aloistr/swisseph/tree/master/windows  
- ルートの `architecture.md`: プロジェクト全体の構造・技術スタック  
- ルートの `readme.md`: 暦ファイルの配置、ライセンス、他言語版（Java/PHP/Perl 等）の案内  

---

*このドキュメントは windows/ 配下のファイルと swephzip.txt、readme の内容に基づいて作成した構成メモです。*
