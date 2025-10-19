# パチスロ筐体 ハードウェア抽象ブロック図

以下はジャグラー系を参考にした一般的パチスロ機の抽象的な内部構成ブロック図です。シミュレータ実装時にクラス/モジュール分割を検討するためのたたき台となります。

## Mermaidブロック図
```mermaid
graph TD
  subgraph Core[メイン基板]
    MCU["メインCPU/MCU<br/>ゲーム制御/抽選"]
    RNG["RNG Engine<br/>擬似乱数状態"]
    MEM[(ROM / RAM / EEPROM)]
    TIMER[System Timer / Interrupt Dispatcher]
    SEC["Security Module<br/>ドア/電圧/基板ID"]
  end

  subgraph ReelSys[リールサブシステム]
    RM1["Reel Motor 1"]
    RS1["Reel Sensor 1"]
    RM2["Reel Motor 2"]
    RS2["Reel Sensor 2"]
    RM3["Reel Motor 3"]
    RS3["Reel Sensor 3"]
  end

  subgraph IOPanel[入力パネル]
    Lever["スタートレバー"]
    Bet["Bet/MaxBet ボタン"]
    StopBtns["各リール停止ボタン"]
    SettingKey["設定キー"]
  end

  subgraph Outputs[出力デバイス]
    GOGO["GOGO! ランプ"]
    PanelLED["パネルLED/表示"]
    Audio["サウンド回路"]
    ReelLamp["リール照明"]
  end

  subgraph Payout[払出システム]
    Hopper["ホッパーモータ"]
    CoinSensor["コインセンサー"]
  end

  subgraph Power[電源/監視]
    PSU["電源基板"]-->VoltMon["電圧モニタ"]
  end

  MCU --> RNG
  MCU --> MEM
  MCU --> TIMER
  MCU --> SEC
  MCU --> ReelSys
  MCU --> IOPanel
  MCU --> Outputs
  MCU --> Payout
  MCU --> Power

  RM1 --> RS1
  RM2 --> RS2
  RM3 --> RS3

  CoinSensor --> Hopper

  IOPanel --> MCU
  ReelSys --> MCU
  Payout --> MCU
  Power --> SEC
  SEC --> MCU

  GOGO --> MCU
  PanelLED --> MCU
  Audio --> MCU
  ReelLamp --> MCU

  style MCU fill:#ffd,stroke:#333
  style RNG fill:#efe,stroke:#393
  style MEM fill:#efe,stroke:#393
  style TIMER fill:#efe,stroke:#393
  style SEC fill:#fee,stroke:#933
  style ReelSys fill:#def,stroke:#339
  style IOPanel fill:#def,stroke:#339
  style Outputs fill:#fdf,stroke:#636
  style Payout fill:#ffd,stroke:#333
  style Power fill:#ddd,stroke:#555
```

## データ/イベントフロー概要
- 入力イベント: IOPanel -> MCU (割り込み/ポーリング)。
- リール位置更新: ReelSensor -> MCU (角度/シンボル)。
- 抽選: MCU -> RNG (乱数取得) -> MCU (役成立判定)。
- ランプ/演出: MCU -> Outputs (優先度制御)。
- 払出: MCU -> Hopper (開始) / CoinSensor -> MCU (枚数フィードバック)。
- セキュリティ: Door/Voltageなど -> SEC -> MCU。基板ID照合は起動時 & 定期チェック。
- 永続データ: MCU <-> MEM (統計/設定)。
- タイマ/割り込み: TIMER -> MCU (周期ハートビート、リール通過イベント)。

## クラス設計候補 (抽象)
| 役割 | クラス案 | 主API例 |
|------|----------|---------|
| リール制御 | ReelMotor, ReelSensor | start(), stop(), get_position() |
| 乱数 | RNGEngine | next_uint32(), uniform(), weighted() |
| 入力 | InputPanel | poll(), events() |
| ランプ/表示 | LampController | set(pattern), priority() |
| 払出 | PayoutController | payout(target), status() |
| セキュリティ | SecurityModule | door_state(), voltage_ok(), board_id() |
| 永続 | MemoryStore | read_stats(), write_stats() |
| タイマ | SystemTimer | now(), schedule(callback, ms) |
| ゲーム進行 | GameCore | spin(), settle(), check_bonus() |

## 今後の拡張ポイント
- 通信IF (例: シリアル, ネットワーク) の抽象追加。
- 演出スクリプトエンジン (ランプ/サウンドパターン DSL)。
- 統計収集モジュール (BigData用途)。
- テストダブル (MockRNG, MockReelMotor) による再現性テスト。

---
シミュレータ実装前に API境界と責務分割をさらに詳細化していきます。追加要望があればコメントください。