# 計算エンジンの構成

## 1. 計算エンジン全体アーキテクチャ

```mermaid
flowchart TB
    subgraph "入力層"
        XML[XML入力データ<br/>33エンティティ]
        WEATHER[気象データ<br/>8760時間×8地域]
        CONST[定数・係数<br/>国交省告示]
        MATERIAL[建材・機器特性<br/>熱伝導率・効率]
    end

    subgraph "計算エンジンコア"
        PARSER[XMLパーサー<br/>入力データ読込]
        VALIDATOR[バリデータ<br/>整合性チェック]
        DISPATCHER[計算ディスパッチャ<br/>設備別に振り分け]
    end

    subgraph "設備別計算モジュール"
        direction TB

        subgraph "第2章: 空調計算モジュール"
            AC_WEATHER[2.2: 気象条件処理]
            AC_SETTING[2.3: 設定温度・スケジュール]
            AC_LOAD[2.4: 室負荷計算<br/>8760h×室数]
            AC_AHU[2.5: 空調機群負荷・消費電力]
            AC_PUMP[2.6: 二次ポンプ消費電力]
            AC_HEAT[2.7: 熱源消費電力]
            AC_TOTAL[2.8: 空調総消費電力]
        end

        V_CALC[第3章: 換気計算モジュール<br/>送風機消費電力]
        L_CALC[第4章: 照明計算モジュール<br/>照明消費電力]
        HW_CALC[第5章: 給湯計算モジュール<br/>給湯消費電力]
        EV_CALC[第6章: 昇降機計算モジュール<br/>昇降機消費電力]
        PV_CALC[第7章: 太陽光発電モジュール<br/>発電量・削減効果]
        CGS_CALC[第8章: コージェネモジュール<br/>発電・排熱利用]
        M_CALC[第9章: その他モジュール<br/>OA機器等]
        ST_CALC[第10章: 基準値計算モジュール<br/>基準一次エネルギー]
    end

    subgraph "結果統合層"
        INTEGRATOR[結果統合<br/>第1章計算式]
        E_AC[E_AC: 空調<br/>MJ/年]
        E_V[E_V: 換気<br/>MJ/年]
        E_L[E_L: 照明<br/>MJ/年]
        E_HW[E_HW: 給湯<br/>MJ/年]
        E_EV[E_EV: 昇降機<br/>MJ/年]
        E_PV[E_PV: 太陽光<br/>MJ/年]
        E_CGS[E_CGS: コージェネ<br/>MJ/年]
        E_M[E_M: その他<br/>MJ/年]
        E_T[E_T: 設計値<br/>GJ/年]
        E_ST[E_ST: 基準値<br/>GJ/年]
    end

    subgraph "出力層"
        FORMATTER[結果フォーマッタ]
        REPORT[評価レポート生成]
        DETAIL[設備別内訳]
        GRAPH[グラフ・図表]
        JSON_OUT[JSON出力]
        PDF_OUT[PDF出力]
    end

    XML --> PARSER
    WEATHER --> PARSER
    CONST --> PARSER
    MATERIAL --> PARSER

    PARSER --> VALIDATOR
    VALIDATOR --> DISPATCHER

    DISPATCHER --> AC_WEATHER
    DISPATCHER --> V_CALC
    DISPATCHER --> L_CALC
    DISPATCHER --> HW_CALC
    DISPATCHER --> EV_CALC
    DISPATCHER --> PV_CALC
    DISPATCHER --> CGS_CALC
    DISPATCHER --> M_CALC
    DISPATCHER --> ST_CALC

    AC_WEATHER --> AC_SETTING
    AC_SETTING --> AC_LOAD
    AC_LOAD --> AC_AHU
    AC_AHU --> AC_PUMP
    AC_PUMP --> AC_HEAT
    AC_HEAT --> AC_TOTAL

    AC_TOTAL --> E_AC
    V_CALC --> E_V
    L_CALC --> E_L
    HW_CALC --> E_HW
    EV_CALC --> E_EV
    PV_CALC --> E_PV
    CGS_CALC --> E_CGS
    M_CALC --> E_M

    E_AC --> INTEGRATOR
    E_V --> INTEGRATOR
    E_L --> INTEGRATOR
    E_HW --> INTEGRATOR
    E_EV --> INTEGRATOR
    E_PV --> INTEGRATOR
    E_CGS --> INTEGRATOR
    E_M --> INTEGRATOR

    ST_CALC --> E_ST

    INTEGRATOR --> E_T
    E_T --> FORMATTER
    E_ST --> FORMATTER

    FORMATTER --> REPORT
    FORMATTER --> DETAIL
    FORMATTER --> GRAPH
    FORMATTER --> JSON_OUT
    FORMATTER --> PDF_OUT

    style XML fill:#e1ffe1
    style PARSER fill:#e1f5ff
    style VALIDATOR fill:#e1f5ff
    style DISPATCHER fill:#ffd700
    style AC_LOAD fill:#ffe1e1
    style INTEGRATOR fill:#ffd700
    style E_T fill:#32cd32
    style E_ST fill:#32cd32
    style REPORT fill:#90ee90
```

## 2. 空調計算モジュールの詳細構成

```mermaid
flowchart TD
    subgraph "入力"
        INPUT_AC[XML: 空調システムデータ]
        INPUT_WEATHER[気象データ]
        INPUT_CONST[定数・係数]
    end

    subgraph "2.2: 気象条件処理"
        W1[外気温度取得<br/>θ_oa,d,t]
        W2[絶対湿度取得<br/>X_oa,d,t]
        W3[日射量取得<br/>S_dsr,d,t, S_isr,d,t]
        W4[外気エンタルピー計算]
        W5[日平均温度計算]
    end

    subgraph "2.3: 設定温度・スケジュール"
        S1[室用途別設定温度]
        S2[運転スケジュール]
        S3[外気導入量]
    end

    subgraph "2.4: 室負荷計算"
        L1[外壁貫流熱計算]
        L2[窓貫流熱計算]
        L3[日射熱取得計算]
        L4[内部発熱計算]
        L5[換気熱負荷計算]
        L6[室負荷統合<br/>Q_AC,r,d,t]
        L7[日積算室負荷]
    end

    subgraph "2.5: 空調機群"
        A1[空調機群への負荷集計]
        A2[外気負荷加算]
        A3[負荷率帯分類<br/>11区分]
        A4[外気温帯分類]
        A5[空調機消費電力<br/>負荷率帯×外気温帯]
    end

    subgraph "2.6: 二次ポンプ"
        P1[ポンプ負荷計算]
        P2[台数制御補正]
        P3[回転数制御補正]
        P4[ポンプ消費電力]
    end

    subgraph "2.7: 熱源"
        H1[熱源負荷計算]
        H2[負荷率帯別効率適用]
        H3[外気温帯補正]
        H4[台数制御補正]
        H5[蓄熱運転補正]
        H6[熱源消費電力]
    end

    subgraph "2.8: 空調総消費電力"
        T1[空調機消費電力]
        T2[ポンプ消費電力]
        T3[熱源消費電力]
        T4[補機消費電力]
        T5[空調総消費電力<br/>E_AC]
    end

    INPUT_AC --> S1
    INPUT_WEATHER --> W1
    INPUT_WEATHER --> W2
    INPUT_WEATHER --> W3
    INPUT_CONST --> L1

    W1 --> W4
    W2 --> W4
    W3 --> L3
    W1 --> W5

    W4 --> L5
    W5 --> L1
    S1 --> L6
    S2 --> L7
    S3 --> L5

    L1 --> L6
    L2 --> L6
    L3 --> L6
    L4 --> L6
    L5 --> L6
    L6 --> L7

    L7 --> A1
    A1 --> A2
    A2 --> A3
    A3 --> A4
    A4 --> A5

    A5 --> P1
    P1 --> P2
    P2 --> P3
    P3 --> P4

    P4 --> H1
    H1 --> H2
    H2 --> H3
    H3 --> H4
    H4 --> H5
    H5 --> H6

    A5 --> T1
    P4 --> T2
    H6 --> T3
    T1 --> T4
    T2 --> T4
    T3 --> T4
    T4 --> T5

    style INPUT_AC fill:#e1ffe1
    style INPUT_WEATHER fill:#ffffcc
    style L6 fill:#ffe1e1
    style A3 fill:#ffd700
    style T5 fill:#32cd32
```

## 3. 計算モジュール間のデータフロー

```mermaid
flowchart LR
    subgraph "前処理"
        PARSE[XMLパース]
        VALIDATE[バリデーション]
    end

    subgraph "並列計算可能"
        direction TB
        AC[空調計算]
        V[換気計算]
        L[照明計算]
        HW[給湯計算]
        EV[昇降機計算]
        PV[太陽光計算]
    end

    subgraph "依存計算"
        CGS[コージェネ計算]
        M[その他計算]
        ST[基準値計算]
    end

    subgraph "後処理"
        INTEGRATE[結果統合]
        FORMAT[フォーマット]
    end

    PARSE --> VALIDATE
    VALIDATE --> AC
    VALIDATE --> V
    VALIDATE --> L
    VALIDATE --> HW
    VALIDATE --> EV
    VALIDATE --> PV

    AC -.排熱負荷.-> CGS
    HW -.給湯負荷.-> CGS
    VALIDATE --> M
    VALIDATE --> ST

    AC --> INTEGRATE
    V --> INTEGRATE
    L --> INTEGRATE
    HW --> INTEGRATE
    EV --> INTEGRATE
    PV --> INTEGRATE
    CGS --> INTEGRATE
    M --> INTEGRATE
    ST --> INTEGRATE

    INTEGRATE --> FORMAT

    style VALIDATE fill:#e1f5ff
    style AC fill:#ffe1e1
    style V fill:#ffe1e1
    style L fill:#ffe1e1
    style HW fill:#ffe1e1
    style EV fill:#ffe1e1
    style PV fill:#90ee90
    style CGS fill:#ffd700
    style INTEGRATE fill:#ffd700
```

## 4. 負荷計算エンジンの詳細

### 4.1 室負荷計算エンジン

```mermaid
flowchart TD
    START[室rの時刻tの負荷計算開始]

    subgraph "外壁貫流熱"
        EW1[外壁リスト取得]
        EW2[外壁面積 A_env]
        EW3[熱貫流率 U]
        EW4[外気温度 θ_oa,t]
        EW5[室温 θ_r,t]
        EW6[Q_env = Σ U × A × ΔT]
    end

    subgraph "窓貫流熱"
        WD1[窓リスト取得]
        WD2[窓面積 A_win]
        WD3[窓熱貫流率 U_win]
        WD4[Q_win = Σ U_win × A_win × ΔT]
    end

    subgraph "日射熱取得"
        SR1[窓方位取得]
        SR2[日射量 S_dsr, S_isr]
        SR3[日射熱取得率 η]
        SR4[日よけ効果係数]
        SR5[Q_sol = Σ A_win × η × S × 係数]
    end

    subgraph "内部発熱"
        IH1[人体発熱<br/>在室人数×発熱量]
        IH2[照明発熱<br/>照明消費電力]
        IH3[OA機器発熱<br/>原単位×面積]
        IH4[Q_int = 人体 + 照明 + OA]
    end

    subgraph "換気熱負荷"
        VT1[外気導入量 V_oa]
        VT2[外気エンタルピー h_oa]
        VT3[室内エンタルピー h_r]
        VT4[Q_vent = ρ × c × V_oa × ΔT]
    end

    subgraph "室負荷統合"
        SUM[Q_AC,r,t = Q_env + Q_win + Q_sol + Q_int + Q_vent]
        COOL{冷房期?}
        HEAT{暖房期?}
        RESULT_C[冷房負荷]
        RESULT_H[暖房負荷]
        RESULT_0[負荷なし]
    end

    START --> EW1
    START --> WD1
    START --> SR1
    START --> IH1
    START --> VT1

    EW1 --> EW2 --> EW3 --> EW4 --> EW5 --> EW6
    WD1 --> WD2 --> WD3 --> WD4
    SR1 --> SR2 --> SR3 --> SR4 --> SR5
    IH1 --> IH2 --> IH3 --> IH4
    VT1 --> VT2 --> VT3 --> VT4

    EW6 --> SUM
    WD4 --> SUM
    SR5 --> SUM
    IH4 --> SUM
    VT4 --> SUM

    SUM --> COOL
    COOL -->|Yes| RESULT_C
    COOL -->|No| HEAT
    HEAT -->|Yes| RESULT_H
    HEAT -->|No| RESULT_0

    style START fill:#e1f5ff
    style SUM fill:#ffd700
    style RESULT_C fill:#ff6347
    style RESULT_H fill:#4169e1
    style RESULT_0 fill:#d3d3d3
```

### 4.2 負荷率帯分類エンジン

```mermaid
flowchart TD
    INPUT[8760時間の負荷データ<br/>Q_t [W]]
    RATED[定格能力 Q_rated [W]]

    CALC[各時刻tの負荷率計算<br/>α_t = Q_t / Q_rated]

    CLASSIFY[11区分への分類]

    subgraph "負荷率帯"
        B0[0-10%]
        B1[10-20%]
        B2[20-30%]
        B3[30-40%]
        B4[40-50%]
        B5[50-60%]
        B6[60-70%]
        B7[70-80%]
        B8[80-90%]
        B9[90-100%]
        B10[100%以上]
    end

    COUNT[各区分の時間数集計<br/>T_hr,n [h]]

    TEMP[外気温帯も同時に分類]

    MATRIX[負荷率帯×外気温帯<br/>の2次元マトリクス]

    INPUT --> CALC
    RATED --> CALC
    CALC --> CLASSIFY

    CLASSIFY --> B0
    CLASSIFY --> B1
    CLASSIFY --> B2
    CLASSIFY --> B3
    CLASSIFY --> B4
    CLASSIFY --> B5
    CLASSIFY --> B6
    CLASSIFY --> B7
    CLASSIFY --> B8
    CLASSIFY --> B9
    CLASSIFY --> B10

    B0 --> COUNT
    B1 --> COUNT
    B2 --> COUNT
    B3 --> COUNT
    B4 --> COUNT
    B5 --> COUNT
    B6 --> COUNT
    B7 --> COUNT
    B8 --> COUNT
    B9 --> COUNT
    B10 --> COUNT

    COUNT --> TEMP
    TEMP --> MATRIX

    style INPUT fill:#e1ffe1
    style CALC fill:#e1f5ff
    style CLASSIFY fill:#ffd700
    style MATRIX fill:#32cd32
```

## 5. エネルギー消費量計算エンジン

### 5.1 空調機消費電力計算

```mermaid
flowchart TD
    LOAD_BAND[負荷率帯n]
    TEMP_BAND[外気温帯m]
    HOURS[時間数 T_hr,n,m]

    CAPACITY[定格能力 Q_rated]
    ALPHA[負荷率 α_n]
    ACTUAL_LOAD[実負荷 Q_n = α_n × Q_rated]

    COEFF[性能係数取得<br/>機器タイプ・負荷率・外気温帯]

    POWER_SUPPLY[給気ファン消費電力<br/>P_supply,n,m]
    POWER_RETURN[還気ファン消費電力<br/>P_return,n,m]
    POWER_OA[外気ファン消費電力<br/>P_oa,n,m]
    POWER_EXHAUST[排気ファン消費電力<br/>P_exhaust,n,m]

    CONTROL_CAV{制御方式<br/>CAV?}
    CONTROL_VAV[VAV制御補正]

    HEX{全熱交<br/>あり?}
    HEX_POWER[全熱交消費電力]

    TOTAL[空調機消費電力<br/>P_AHU,n,m]
    ANNUAL[年間消費電力<br/>E_AHU = Σ P_AHU,n,m × T_hr,n,m]

    LOAD_BAND --> ALPHA
    TEMP_BAND --> COEFF
    CAPACITY --> ACTUAL_LOAD
    ALPHA --> ACTUAL_LOAD

    ACTUAL_LOAD --> POWER_SUPPLY
    COEFF --> POWER_SUPPLY
    COEFF --> POWER_RETURN
    COEFF --> POWER_OA
    COEFF --> POWER_EXHAUST

    POWER_SUPPLY --> CONTROL_CAV
    CONTROL_CAV -->|Yes| TOTAL
    CONTROL_CAV -->|No| CONTROL_VAV
    CONTROL_VAV --> TOTAL

    POWER_RETURN --> TOTAL
    POWER_OA --> TOTAL
    POWER_EXHAUST --> TOTAL

    TOTAL --> HEX
    HEX -->|Yes| HEX_POWER
    HEX -->|No| ANNUAL
    HEX_POWER --> ANNUAL

    HOURS --> ANNUAL

    style LOAD_BAND fill:#e1f5ff
    style COEFF fill:#ffd700
    style TOTAL fill:#ffe1e1
    style ANNUAL fill:#32cd32
```

### 5.2 熱源消費電力計算

```mermaid
flowchart TD
    LOAD[熱源負荷<br/>Q_hs,n [kW]]
    TYPE[熱源機種<br/>Type]
    RATED_CAP[定格能力<br/>Q_hs,rated [kW]]
    LOAD_RATIO[負荷率<br/>α_hs,n = Q_hs,n / Q_hs,rated]

    CURVE[効率曲線取得<br/>COP_n = f(α_hs,n)]

    TEMP_BAND[外気温帯m]
    TEMP_COEFF[外気温帯補正係数<br/>f_temp,m]

    STORAGE{蓄熱<br/>あり?}
    STORAGE_MODE[蓄熱モード<br/>追掛/氷蓄熱/水蓄熱]
    STORAGE_COEFF[蓄熱運転補正]

    COUNT_CTRL{台数制御<br/>あり?}
    COUNT_COEFF[台数制御補正]

    POWER_MAIN[主機消費電力<br/>P_main,n,m = Q_hs,n / COP × 補正]
    POWER_SUB[補機消費電力<br/>P_sub,n,m]
    POWER_PUMP1[一次ポンプ<br/>P_pump1,n,m]
    POWER_CT_FAN[冷却塔ファン<br/>P_ct_fan,n,m]
    POWER_CT_PUMP[冷却水ポンプ<br/>P_ct_pump,n,m]

    TOTAL_HS[熱源消費電力<br/>P_hs,n,m]
    ANNUAL_HS[年間消費電力<br/>E_HS = Σ P_hs,n,m × T_hr,n,m]

    LOAD --> LOAD_RATIO
    RATED_CAP --> LOAD_RATIO
    TYPE --> CURVE
    LOAD_RATIO --> CURVE

    CURVE --> TEMP_COEFF
    TEMP_BAND --> TEMP_COEFF

    TEMP_COEFF --> STORAGE
    STORAGE -->|Yes| STORAGE_MODE
    STORAGE_MODE --> STORAGE_COEFF
    STORAGE -->|No| COUNT_CTRL
    STORAGE_COEFF --> COUNT_CTRL

    COUNT_CTRL -->|Yes| COUNT_COEFF
    COUNT_CTRL -->|No| POWER_MAIN
    COUNT_COEFF --> POWER_MAIN

    POWER_MAIN --> TOTAL_HS
    POWER_SUB --> TOTAL_HS
    POWER_PUMP1 --> TOTAL_HS
    POWER_CT_FAN --> TOTAL_HS
    POWER_CT_PUMP --> TOTAL_HS

    TOTAL_HS --> ANNUAL_HS

    style LOAD fill:#e1ffe1
    style CURVE fill:#ffd700
    style TEMP_COEFF fill:#e1f5ff
    style TOTAL_HS fill:#ffe1e1
    style ANNUAL_HS fill:#32cd32
```

## 6. 計算エンジンの実装パターン

### 6.1 オブジェクト指向設計（推奨）

```mermaid
classDiagram
    class Building {
        +string Name
        +string Region
        +List~Room~ Rooms
        +List~AirConditioningZone~ ACZones
        +calculate() EnergyResult
    }

    class Room {
        +int Floor
        +string Name
        +string BuildingType
        +string RoomType
        +decimal Area
        +calculateLoad() Load
    }

    class AirConditioningZone {
        +int Floor
        +string Name
        +List~Room~ Rooms
        +EnvelopeSet Envelope
        +AirHandlingUnitSet AHU
        +calculateEnergy() Energy
    }

    class LoadCalculator {
        +calculateWallLoad() decimal
        +calculateWindowLoad() decimal
        +calculateSolarLoad() decimal
        +calculateInternalLoad() decimal
        +calculateVentilationLoad() decimal
    }

    class AirHandlingUnitSet {
        +string Name
        +List~AirHandlingUnit~ Units
        +HeatSourceSet CoolingHeatSource
        +HeatSourceSet HeatingHeatSource
        +calculateEnergy() Energy
    }

    class HeatSourceSet {
        +string Name
        +List~HeatSource~ Units
        +boolean QuantityControl
        +calculateEnergy() Energy
    }

    class EnergyCalculator {
        +calculateAC() decimal
        +calculateVentilation() decimal
        +calculateLighting() decimal
        +calculateHotwater() decimal
        +calculateElevator() decimal
        +integrate() EnergyResult
    }

    class EnergyResult {
        +decimal E_AC
        +decimal E_V
        +decimal E_L
        +decimal E_HW
        +decimal E_EV
        +decimal E_PV
        +decimal E_CGS
        +decimal E_M
        +decimal E_T
        +decimal E_ST
        +decimal BEI
    }

    Building "1" --> "*" Room
    Building "1" --> "*" AirConditioningZone
    AirConditioningZone "*" --> "*" Room
    AirConditioningZone "1" --> "1" AirHandlingUnitSet
    AirHandlingUnitSet "*" --> "1" HeatSourceSet
    LoadCalculator --> Room
    EnergyCalculator --> Building
    EnergyCalculator --> EnergyResult
```

### 6.2 関数型設計（代替案）

```mermaid
flowchart LR
    subgraph "純粋関数"
        F1[parseXML<br/>XML → Building]
        F2[validateBuilding<br/>Building → Validation]
        F3[calculateLoad<br/>Room × Weather → Load]
        F4[classifyLoadBand<br/>Load → LoadBand]
        F5[calculateEnergy<br/>LoadBand → Energy]
        F6[integrateEnergy<br/>List~Energy~ → Total]
    end

    subgraph "データ構造"
        D1[Building<br/>immutable record]
        D2[Room<br/>immutable record]
        D3[Load<br/>immutable record]
        D4[LoadBand<br/>immutable record]
        D5[Energy<br/>immutable record]
        D6[EnergyResult<br/>immutable record]
    end

    F1 --> D1
    D1 --> F2
    D1 --> D2
    D2 --> F3
    F3 --> D3
    D3 --> F4
    F4 --> D4
    D4 --> F5
    F5 --> D5
    D5 --> F6
    F6 --> D6

    style F1 fill:#e1f5ff
    style F3 fill:#ffe1e1
    style F6 fill:#ffd700
    style D6 fill:#32cd32
```

## 7. パフォーマンス最適化戦略

### 7.1 並列計算の可能性

```mermaid
flowchart TD
    START[計算開始]

    subgraph "並列実行可能"
        P1[空調計算<br/>スレッド1]
        P2[換気計算<br/>スレッド2]
        P3[照明計算<br/>スレッド3]
        P4[給湯計算<br/>スレッド4]
        P5[昇降機計算<br/>スレッド5]
        P6[太陽光計算<br/>スレッド6]
    end

    BARRIER[同期バリア]

    subgraph "順次実行"
        S1[コージェネ計算]
        S2[その他計算]
        S3[基準値計算]
    end

    INTEGRATE[結果統合]
    END[計算完了]

    START --> P1
    START --> P2
    START --> P3
    START --> P4
    START --> P5
    START --> P6

    P1 --> BARRIER
    P2 --> BARRIER
    P3 --> BARRIER
    P4 --> BARRIER
    P5 --> BARRIER
    P6 --> BARRIER

    BARRIER --> S1
    S1 --> S2
    S2 --> S3
    S3 --> INTEGRATE
    INTEGRATE --> END

    style START fill:#e1f5ff
    style BARRIER fill:#ffd700
    style INTEGRATE fill:#32cd32
    style END fill:#90ee90
```

### 7.2 キャッシュ戦略

| データ | キャッシュ対象 | 理由 |
|-------|------------|------|
| 気象データ | メモリキャッシュ | 8760時間×8地域＝70080レコード、頻繁にアクセス |
| 定数・係数 | メモリキャッシュ | 変更頻度低い、サイズ小 |
| 建材物性値 | メモリキャッシュ | 変更頻度低い |
| 室負荷計算結果 | ディスクキャッシュ | 8760時間×室数、再計算コスト高 |
| 負荷率帯分類結果 | メモリキャッシュ | 小サイズ、頻繁に参照 |

### 7.3 計算時間見積もり（中規模建物）

| 計算フェーズ | 時間 | 説明 |
|-----------|-----|------|
| XMLパース | 0.1秒 | 500KBのXMLファイル |
| バリデーション | 0.2秒 | 整合性チェック |
| 気象データ読込 | 0.5秒 | 70080レコード |
| 空調負荷計算 | 10～30秒 | 8760h×50室＝438,000計算 |
| 空調機消費電力 | 2～5秒 | 負荷率帯×外気温帯×機器数 |
| 熱源消費電力 | 2～5秒 | 負荷率帯×外気温帯×熱源数 |
| 換気計算 | 1～2秒 | 送風機数×運転時間 |
| 照明計算 | 1～2秒 | 照明機器数×年間点灯時間 |
| 給湯計算 | 1～3秒 | 365日×給湯箇所数 |
| その他計算 | 1秒 | 単純計算 |
| 結果統合 | 0.5秒 | 統合・フォーマット |
| **合計** | **20～50秒** | シングルスレッド実行 |

**並列実行時の見積もり:** 10～20秒（6コア並列の場合）

## 8. エラーハンドリングと例外処理

```mermaid
flowchart TD
    CALC[計算実行]

    CHECK1{入力データ<br/>有効?}
    CHECK2{計算結果<br/>妥当?}
    CHECK3{収束<br/>判定OK?}

    ERROR1[入力エラー<br/>• 必須項目欠落<br/>• データ型不正<br/>• 範囲外の値]

    ERROR2[計算エラー<br/>• ゼロ除算<br/>• オーバーフロー<br/>• 数値不安定]

    ERROR3[収束エラー<br/>• 反復計算未収束<br/>• 負荷計算バランス不良]

    WARNING[警告<br/>• 推奨範囲外の値<br/>• データ補完実施]

    RETRY[リトライ<br/>• パラメータ調整<br/>• 初期値変更]

    SUCCESS[計算成功]
    REPORT[結果出力]

    CALC --> CHECK1
    CHECK1 -->|No| ERROR1
    CHECK1 -->|Yes| CHECK2
    CHECK2 -->|No| ERROR2
    CHECK2 -->|Yes| CHECK3
    CHECK3 -->|No| ERROR3
    CHECK3 -->|Yes| WARNING
    WARNING --> SUCCESS
    SUCCESS --> REPORT

    ERROR2 --> RETRY
    ERROR3 --> RETRY
    RETRY --> CALC

    style ERROR1 fill:#dc143c
    style ERROR2 fill:#dc143c
    style ERROR3 fill:#dc143c
    style WARNING fill:#ffa500
    style SUCCESS fill:#32cd32
```

## 9. まとめ

Webpro計算エンジンは以下の特徴を持ちます：

1. **モジュール化**: 設備別に独立した計算モジュール
2. **並列化可能**: 空調・換気・照明等が並列実行可能
3. **スケーラブル**: 室数・機器数に応じて計算量が線形に増加
4. **精密**: 8760時間×負荷率帯×外気温帯の詳細計算
5. **透明性**: すべての計算式が公開仕様に従う
6. **拡張性**: 新規設備カテゴリの追加が容易
7. **高速**: 中規模建物で20～50秒（並列化で10～20秒）
8. **堅牢**: 多段階のエラーチェックと例外処理

このアーキテクチャにより、正確かつ高速な建築物エネルギー性能評価を実現しています。
