# ILiS ME BMS ダッシュボードアプリ — プロジェクト CLAUDE.md

## プロジェクト概要
- **アプリ名**: ILiS ME（仮）
- **目的**: キャンピングカーのサブバッテリー（JBD BMS）の状態を1画面ダッシュボードで表示する
- **ターゲット**: キャンピングカーオーナー（非エンジニア）
- **フレームワーク**: Flutter（Dart）
- **対応OS**: Android（初期）、iOS（将来対応）
- **言語**: 日本語＋英語の2言語対応
- **BLE通信**: JBD BMS と Bluetooth Low Energy で通信
- **BMS接続台数**: まず1台対応で開発。将来的に複数台対応を拡張予定
- **BMS認証**: 認証不要（接続後すぐにデータ取得可能）
- **配布範囲**: まず自社用、将来的に一般配布も検討

## コンテキスト管理（最重要）
- Proプランのため、コンテキストの消費を常に最小限にすること
- 回答は簡潔にし、聞かれていないことまで説明しない
- コードを表示するとき、変更がない部分は省略すること
- ファイル全体を表示せず、変更箇所だけを示すこと
- 長い出力が必要なときは事前に確認すること

## 基本ルール
- 私は非エンジニアです。専門用語を使うときは、必ず簡単な言葉で補足してください
- すべてのやりとりは日本語で行ってください
- コード内のコメントも日本語で書いてください
- 作業を始める前に「何をするか」を簡潔に説明してから実行してください
- 勝手にファイルを削除しないでください。削除が必要な場合は必ず事前に確認してください

## 作業スタイル
- 小さな単位で作業を進め、こまめにgit commitしてください
- commitメッセージは日本語で、何をしたかわかるように書いてください
- エラーが出たら、原因と直し方をわかりやすく説明してください
- 複数のやり方があるときは、最もシンプルな方法を選んでください
- 判断に迷うことがあれば、選択肢を示して私に聞いてください
- Planモードで計画→確認→Actモードで実行の流れ
- フェーズごとに会話を分けてトークン節約
- モデル: opusplan推奨（Planモード=Opus、実行=Sonnet）

## コードの書き方
- シンプルで読みやすいコードを書いてください
- 変数名や関数名は、何をするものか意味がわかる名前にしてください
- 1つの関数は1つの役割だけ持つようにしてください
- 新しいライブラリを追加するときは、なぜ必要かを説明してください

## 安全のためのルール
- 重要な操作（データの削除、設定の変更など）の前には必ず確認してください
- パスワードやAPIキーなどの秘密情報は、コードに直接書かず環境変数を使ってください
- エラーを握りつぶさないでください（問題を隠さず、ちゃんと報告する書き方にしてください）

## Gitの使い方
- commitは作業の区切りごとにこまめに行ってください
- 意味のある単位でcommitしてください（「いろいろ修正」のようなまとめ方はしないでください）
- 新しい機能を作るときはブランチを作ることを提案してください

---

## 技術スタック

### Flutter
- **言語**: Dart
- **BLEライブラリ**: flutter_blue_plus（推奨）
- **状態管理**: Provider または Riverpod（要検討）
- **多言語対応**: flutter_localizations + intl

### 開発環境
- Mac (Intel)
- VSCode + Claude Code
- Android実機テスト（初期）
- テスト環境: flutter test

---

## ダッシュボードUI仕様

### 画面構成
- **メイン画面**: 1画面ダッシュボード（縦横両対応）
- **BLE接続画面**: デバイススキャン・選択・接続
- **言語設定画面**: 日本語/英語切替

### ダッシュボードレイアウト（縦画面）
上から順に:

1. **SOCエリア（最も大きく目立つ）**
   - バッテリー残量を大きな円グラフまたはバッテリーアイコンで表示
   - 数値（%）も大きく表示
   - 色変化: 60〜100%=緑、20〜59%=黄、0〜19%=赤
   - データソース: `soc`

2. **ステータスエリア**
   - 充電中 / 放電中 / 待機中 を大きく表示
   - 充電中=緑、放電中=オレンジ、待機中=グレー
   - 判定: electricity > 0 → 充電、< 0 → 放電、== 0 → 待機
   - データソース: `electricity`, `fet`

3. **電流・電力エリア**
   - 充放電電流（A）: `electricity`
   - 電力（W）: `power`
   - パック総電圧（V）: `totalVoltage`
   - 残容量（Ah）: `surplusCapacity`

4. **セル電圧エリア**
   - 各セル電圧一覧: `voltageList`
   - 最高/最低/平均: `highestVoltage`, `lowestVoltage`, `averageVoltage`
   - 電圧差: `dropoutVoltage`（0.05V超で警告色）

5. **温度エリア**
   - 温度一覧: `temperaturesList`
   - 色変化: 0℃以下=青、1〜40℃=緑、41℃以上=赤

6. **Bluetooth接続ステータス**
   - 接続デバイス名、接続/切断ボタン
   - 未接続時はスキャン画面への遷移ボタン

### ダッシュボードレイアウト（横画面）
- 左半分: SOC + ステータス + 電流・電力
- 右半分: セル電圧 + 温度

### デザイン方針
- フォントは大きく（運転席からチラ見できるサイズ）
- 背景はダーク系（車内での視認性、夜間の眩しさ防止）
- カラーは最小限（緑=正常、黄=注意、赤=異常）
- アイコン活用、テキスト最小限

---

## BMS通信プロトコル仕様

### BLE接続情報

| 項目 | 値 |
|---|---|
| サービスUUID | `0000FF00-0000-1000-8000-00805F9B34FB` |
| Notify UUID | `0000FF01-0000-1000-8000-00805F9B34FB` |
| Write UUID | `0000FF02-0000-1000-8000-00805F9B34FB` |

### BLE接続シーケンス
1. BLEスキャン開始
2. デバイス発見 → ユーザーが選択
3. BLE接続確立
4. サービス取得（UUID: FF00）
5. キャラクタリスティック取得（FF01=Notify, FF02=Write）
6. Notify購読開始（FF01）
7. コマンド書き込み（FF02）→ レスポンス受信（FF01）

### パケット構造

**リクエスト:**
```
[0xDD] [0xA5] [CMD] [DATA_LEN] [DATA...] [CRC_HIGH] [CRC_LOW] [0x77]
```
- 0xA5 = 読み取り、0x5A = 書き込み

**レスポンス:**
```
[0xDD] [CMD] [STATUS] [DATA_LEN] [DATA...] [CRC_HIGH] [CRC_LOW] [0x77]
```

### チェックサム計算
```dart
List<int> crcCheckSum(List<int> bytes) {
  // bytes = パケットのslice(2, length-3)
  final sum = bytes.fold(0, (acc, val) => acc + val);
  final hex = sum.toRadixString(16).padLeft(4, '0').toUpperCase();
  return [int.parse(hex.substring(0, 2), radix: 16),
          int.parse(hex.substring(2, 4), radix: 16)];
}
```

### データ取得コマンド

| コマンド | バイト列 | 内容 |
|---------|---------|------|
| DDA5_03 | DD A5 03 00 FF FD 77 | 総電圧・電流・SOC・温度・ステータス |
| DDA5_04 | DD A5 04 00 FF FC 77 | セル電圧一覧 |
| DDA5_05 | DD A5 05 00 FF FB 77 | ハードウェアバージョン |
| DDA5_06 | DD A5 06 00 FF FA 77 | 現在時刻（BCD形式） |

### DDA5_03 レスポンスのパース仕様（最重要）

```dart
Map<String, dynamic> parseDDA503(List<int> e) {
  // スケール判定: e[24]のbit7
  final scale = ((e[24] >> 7) & 1) == 1 ? 10 : 100;

  // 総電圧 (V)
  final totalVoltage = ((e[4] << 8) + e[5]) / 100;

  // 電流 (A) — 正=充電、負=放電
  var currentRaw = (e[6] << 8) + e[7];
  final current = currentRaw > 32768
      ? (currentRaw - 65536) / scale
      : currentRaw / scale;

  // 電力 (W)
  final power = totalVoltage * current;

  // SOC (%)
  final soc = e[23];

  // FETステータス
  // bit0=充電SW, bit1=放電SW, bit4=加熱中, bit6=ファクトリーモード
  final fet = e[24];

  // 直列セル数
  final cellCount = e[25];

  // NTC温度センサー数
  final ntcCount = e[26];

  // 温度 (℃) — ケルビン変換
  final temps = <double>[];
  for (var i = 0; i < ntcCount; i++) {
    final offset = 27 + i * 2;
    final raw = (e[offset] << 8) + e[offset + 1];
    temps.add((raw - 2731) / 10);
  }

  // 保護ステータス（ビットマスク）
  final protectStatus = (e[20] << 8) + e[21];

  // 残量 (Ah)
  final surplusRaw = (e[8] << 8) + e[9];
  final surplusCapacity = surplusRaw / scale;

  // 満充電容量 (Ah)
  final fullCapRaw = (e[10] << 8) + e[11];
  final fullCapacity = fullCapRaw / scale;

  // SOH (%) — 計算値
  final soh = fullCapacity > 0
      ? (surplusCapacity / fullCapacity * 100).round()
      : 0;

  return {
    'totalVoltage': totalVoltage,
    'current': current,
    'power': power,
    'soc': soc,
    'fet': fet,
    'cellCount': cellCount,
    'temps': temps,
    'protectStatus': protectStatus,
    'surplusCapacity': surplusCapacity,
    'fullCapacity': fullCapacity,
    'soh': soh,
  };
}
```

### DDA5_04 レスポンスのパース仕様（セル電圧）

```dart
Map<String, dynamic> parseDDA504(List<int> e) {
  final dataLen = e[3];
  final voltages = <double>[];
  for (var i = 4; i < dataLen + 4; i += 2) {
    final raw = (e[i] << 8) + e[i + 1]; // mV
    voltages.add(raw / 1000);             // V
  }

  final highest = voltages.reduce(max);
  final lowest = voltages.reduce(min);
  final average = voltages.reduce((a, b) => a + b) / voltages.length;
  final dropout = highest - lowest;

  return {
    'voltageList': voltages,
    'highestVoltage': highest,
    'lowestVoltage': lowest,
    'averageVoltage': average,
    'dropoutVoltage': dropout,
  };
}
```

### ポーリング
- 推奨間隔: 5秒
- BLE接続完了後にTimerでポーリング開始
- 各ポーリングで DDA503 + DDA504 を送信・パース
- BLE切断時にTimer停止

```dart
Timer? _pollingTimer;

void startPolling() {
  _pollingTimer = Timer.periodic(Duration(seconds: 5), (_) async {
    final raw03 = await sendCommand(CMD_DDA503);
    final data03 = parseDDA503(raw03);

    final raw04 = await sendCommand(CMD_DDA504);
    final data04 = parseDDA504(raw04);

    // UI更新
    setState(() {
      _batteryData = data03;
      _cellData = data04;
    });
  });
}

void stopPolling() {
  _pollingTimer?.cancel();
}
```

### 認証
- **自社BMSは認証不要**（接続後すぐにデータ取得可能 — 確認済み）
- 認証ロジック（appkey認証、パスワード認証）は将来の他社BMS対応時に検討

### 制御コマンド（参考）

| コマンド | バイト列 | 内容 |
|---------|---------|------|
| DD5A_E3 | DD 5A E3 07 [BCD時刻] ... 77 | 時刻設定 |
| DD5A_FB | DD 5A FB 02 [p1] [p2] ... 77 | MOSFET制御 |

---

## 開発フェーズ計画

### フェーズ1: プロジェクトセットアップ
- Flutter環境構築（Flutter SDK、Android SDK）
- Flutter プロジェクト作成
- 基本ディレクトリ構成
- flutter_blue_plus 等の依存追加
- 多言語対応の基盤（日本語/英語）
- git初期化・初回コミット

### フェーズ2: BLE通信実装
- BLEスキャン・接続画面
- DDA503 / DDA504 コマンド送信
- レスポンスパース
- 5秒ポーリング
- BLE切断・再接続ハンドリング

### フェーズ3: ダッシュボードUI
- SOC表示（円グラフ、色変化）
- ステータス表示（充電/放電/待機）
- 電流・電力・電圧表示
- セル電圧一覧
- 温度表示（色変化）
- ダークテーマ
- 縦横レスポンシブ対応

### フェーズ4: 仕上げ
- アプリアイコン（ILiS MEブランドロゴ — デザイナーからデータ受領予定）
- エラーハンドリング（BLE切断、タイムアウト等）
- 実機テスト・バグ修正
- リリースビルド

### 将来フェーズ: 複数台対応
- 複数BMS同時接続（BLE同時接続管理）
- BMS切り替えUI（タブ or リスト）
- デバイスごとのダッシュボード表示

---

## 過去プロジェクトからの教訓

### ローカライズプロジェクト（成功）
- JBD BMS apk（中国製）の日本語完全ローカライズを完了
- UniApp + vue-i18n の仕組みを解明し、1,123キーの翻訳を適用
- 「仕組みの理解」が作業の9割

### 得られた最大の資産
- BMS通信プロトコルの完全な仕様書（上記セクション）
- BLEサービス/キャラクタリスティックUUID
- コマンドバイト列とレスポンスパース仕様
- これにより、新規アプリでBMS通信をゼロから実装可能
