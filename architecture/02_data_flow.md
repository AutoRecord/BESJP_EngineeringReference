# データフロー図

## 1. 全体データフロー

```mermaid
flowchart TD
    subgraph "入力フェーズ"
        F0[様式0: 基本情報<br/>建物名、地域、階数]
        F1[様式1: 室仕様<br/>室面積、用途、階高]
        F2[様式2-1～2-7: 空調<br/>ゾーン、外皮、熱源、ポンプ、空調機]
        F3[様式3-1～3-3: 換気<br/>換気室、送風機]
        F4[様式4: 照明<br/>照明器具]
        F5[様式5-1～5-2: 給湯<br/>給湯室、給湯機器]
        F6[様式6: 昇降機<br/>エレベータ仕様]
        F7[様式7-1～7-3: 発電・コージェネ<br/>太陽光、コージェネ]
    end

    subgraph "変換フェーズ"
        CONV[CSV→XML変換<br/>csv2xml仕様に従う]

        subgraph "変換ルール"
            R1[common.adoc<br/>建物用途・室用途マッピング]
            R2[basic_information.adoc<br/>Model要素へ変換]
            R3[room.adoc<br/>Room要素へ変換]
            R4[空調関連変換<br/>8ファイル]
            R5[換気関連変換<br/>3ファイル]
            R6[照明・給湯変換<br/>3ファイル]
            R7[その他変換<br/>4ファイル]
        end
    end

    subgraph "統合データ"
        XML[XML形式統合データ]

        subgraph "XMLスキーマ (33エンティティ)"
            E1[Model: 基本情報]
            E2[Room: 室仕様]
            E3[AirConditioningZone等<br/>空調17エンティティ]
            E4[VentilationRoom等<br/>換気5エンティティ]
            E5[LightingRoom等<br/>照明2エンティティ]
            E6[HotwaterRoom等<br/>給湯3エンティティ]
            E7[Elevator: 昇降機]
            E8[PhotovoltaicGeneration等<br/>発電・コージェネ4エンティティ]
        end
    end

    subgraph "計算フェーズ"
        subgraph "外部データ読込"
            W[気象データ<br/>8760時間×8地域]
            C[定数・係数<br/>国交省告示]
            M[建材・機器特性<br/>熱伝導率、効率等]
        end

        subgraph "計算エンジン"
            direction TB
            CALC[計算エンジン統括]

            subgraph "設備別計算モジュール"
                AC_CALC[空調計算<br/>第2章]
                V_CALC[換気計算<br/>第3章]
                L_CALC[照明計算<br/>第4章]
                HW_CALC[給湯計算<br/>第5章]
                EV_CALC[昇降機計算<br/>第6章]
                PV_CALC[太陽光発電<br/>第7章]
                CGS_CALC[コージェネ<br/>第8章]
                M_CALC[その他<br/>第9章]
                ST_CALC[基準値<br/>第10章]
            end
        end
    end

    subgraph "出力フェーズ"
        subgraph "計算結果"
            E_AC[E_AC: 空調エネルギー<br/>MJ/年]
            E_V[E_V: 換気エネルギー<br/>MJ/年]
            E_L[E_L: 照明エネルギー<br/>MJ/年]
            E_HW[E_HW: 給湯エネルギー<br/>MJ/年]
            E_EV[E_EV: 昇降機エネルギー<br/>MJ/年]
            E_PV[E_PV: 太陽光削減<br/>MJ/年]
            E_CGS[E_CGS: コージェネ削減<br/>MJ/年]
            E_M[E_M: その他エネルギー<br/>MJ/年]
        end

        SUM[統合計算<br/>第1章]
        E_T[E_T: 設計一次エネルギー<br/>GJ/年]
        E_ST[E_ST: 基準一次エネルギー<br/>GJ/年]

        REPORT[評価レポート<br/>設計値/基準値<br/>設備別内訳]
    end

    F0 --> CONV
    F1 --> CONV
    F2 --> CONV
    F3 --> CONV
    F4 --> CONV
    F5 --> CONV
    F6 --> CONV
    F7 --> CONV

    CONV --> R1
    CONV --> R2
    CONV --> R3
    CONV --> R4
    CONV --> R5
    CONV --> R6
    CONV --> R7

    R1 --> XML
    R2 --> E1
    R3 --> E2
    R4 --> E3
    R5 --> E4
    R6 --> E5
    R6 --> E6
    R7 --> E7
    R7 --> E8

    E1 --> XML
    E2 --> XML
    E3 --> XML
    E4 --> XML
    E5 --> XML
    E6 --> XML
    E7 --> XML
    E8 --> XML

    XML --> CALC
    W --> CALC
    C --> CALC
    M --> CALC

    CALC --> AC_CALC
    CALC --> V_CALC
    CALC --> L_CALC
    CALC --> HW_CALC
    CALC --> EV_CALC
    CALC --> PV_CALC
    CALC --> CGS_CALC
    CALC --> M_CALC
    CALC --> ST_CALC

    AC_CALC --> E_AC
    V_CALC --> E_V
    L_CALC --> E_L
    HW_CALC --> E_HW
    EV_CALC --> E_EV
    PV_CALC --> E_PV
    CGS_CALC --> E_CGS
    M_CALC --> E_M

    E_AC --> SUM
    E_V --> SUM
    E_L --> SUM
    E_HW --> SUM
    E_EV --> SUM
    E_PV --> SUM
    E_CGS --> SUM
    E_M --> SUM

    ST_CALC --> E_ST

    SUM --> E_T
    E_T --> REPORT
    E_ST --> REPORT

    style F0 fill:#fff4e1
    style F1 fill:#fff4e1
    style F2 fill:#fff4e1
    style F3 fill:#fff4e1
    style F4 fill:#fff4e1
    style F5 fill:#fff4e1
    style F6 fill:#fff4e1
    style F7 fill:#fff4e1
    style CONV fill:#e1f5ff
    style XML fill:#e1ffe1
    style CALC fill:#ffe1e1
    style SUM fill:#ffe1e1
    style E_T fill:#ffd700
    style E_ST fill:#ffd700
    style REPORT fill:#90ee90
```

## 2. CSV→XML変換の詳細フロー

```mermaid
flowchart LR
    subgraph "CSV入力"
        CSV_ROW[CSVの1行]
    end

    subgraph "変換処理"
        PARSE[列解析]
        MAP[マッピング<br/>csv2xml/*.adoc]
        VALIDATE[バリデーション<br/>• データ型チェック<br/>• 必須項目チェック<br/>• 範囲チェック]
        TRANSFORM[変換<br/>• enum値変換<br/>• 単位換算<br/>• デフォルト値補完]
        GROUP[グループ化<br/>• 複数行 → 1要素<br/>• 例: 外壁11行 → 1つのWallConfigure]
    end

    subgraph "XML出力"
        XML_ELEM[XML要素生成]
        XML_ATTR[属性設定]
        XML_REF[外部キー参照]
    end

    CSV_ROW --> PARSE
    PARSE --> MAP
    MAP --> VALIDATE
    VALIDATE --> TRANSFORM
    TRANSFORM --> GROUP
    GROUP --> XML_ELEM
    XML_ELEM --> XML_ATTR
    XML_ATTR --> XML_REF

    style CSV_ROW fill:#fff4e1
    style MAP fill:#e1f5ff
    style VALIDATE fill:#ffe1e1
    style TRANSFORM fill:#ffe1e1
    style GROUP fill:#ffe1e1
    style XML_ELEM fill:#e1ffe1
```

### 2.1 変換の具体例

**例1: 様式1（室仕様）の変換**

```
CSV入力:
①階, ①室名, ②建物用途, ②室用途, ③室面積, ④階高, ⑤天井高, ⑥計算対象室
1, 事務室1, 事務所, 事務室, 50.0, 3.5, 2.7, ＃

↓ 変換処理

XML出力:
<Room>
  <Floor>1</Floor>
  <Name>事務室1</Name>
  <BuildingType>Office</BuildingType>
  <RoomType>OfficeRoom</RoomType>
  <RoomArea>50.0</RoomArea>
  <FloorHeight>3.5</FloorHeight>
  <RoomHeight>2.7</RoomHeight>
  <AirConditioning>true</AirConditioning>
  <Ventilation>false</Ventilation>
  <Lighting>false</Lighting>
  <Hotwater>false</Hotwater>
</Room>
```

**変換ポイント:**
- 「事務所」 → `BuildingType="Office"` (enum変換)
- 「事務室」 → `RoomType="OfficeRoom"` (enum変換)
- 「＃」記号の有無 → `AirConditioning=true/false` (boolean変換)

**例2: 様式2-2（外壁構成）の変換（11行単位）**

```
CSV入力:
①外壁名称, ②壁の種類, ③熱貫流率
外壁1, 外壁, 0.534
, ,
④建材番号, ⑤建材名称, ⑥厚み, ⑦備考
A101, コンクリート, 0.150,
A201, 断熱材, 0.050,
, , ,
, , ,
(以下空行まで11行)

↓ 変換処理（11行を1つの要素に統合）

XML出力:
<WallConfigure>
  <Name>外壁1</Name>
  <Type>Air</Type>
  <Uvalue>0.534</Uvalue>
  <Material>
    <Layer>1</Layer>
    <Number>A101</Number>
    <Name>コンクリート</Name>
    <Thickness>0.150</Thickness>
  </Material>
  <Material>
    <Layer>2</Layer>
    <Number>A201</Number>
    <Name>断熱材</Name>
    <Thickness>0.050</Thickness>
  </Material>
</WallConfigure>
```

**変換ポイント:**
- 11行単位でグループ化
- 「外壁」 → `Type="Air"` (enum変換)
- 建材層は④が入力されている行のみカウント
- 最大9層まで対応

## 3. 計算エンジンのデータフロー（空調計算の例）

```mermaid
flowchart TD
    subgraph "入力データ"
        XML_IN[XML入力データ]
        WEATHER_IN[気象データ<br/>8760時間]
        CONST_IN[定数データ]
    end

    subgraph "第2章: 空調計算"
        direction TB

        STEP1[2.2: 気象条件処理<br/>外気温・湿度・日射]
        STEP2[2.3: 設定温度・スケジュール<br/>運転時間、設定温度]
        STEP3[2.4: 室負荷計算<br/>8760時間×室数]

        subgraph "室負荷の内訳"
            L1[外壁貫流熱]
            L2[窓貫流熱]
            L3[日射熱取得]
            L4[内部発熱]
            L5[換気熱負荷]
        end

        STEP4[2.5: 空調機群負荷<br/>室負荷の統合]
        STEP5[2.5.3: 負荷率帯分類<br/>11区分×8760時間]
        STEP6[2.5.4～2.5.7: 空調機消費電力<br/>負荷率帯別×外気温帯別]
        STEP7[2.6: 二次ポンプ消費電力<br/>台数制御・回転数制御]
        STEP8[2.7: 熱源消費電力<br/>負荷率帯別効率曲線]
        STEP9[2.8: 空調総消費電力<br/>E_AC 統合]
    end

    subgraph "出力"
        E_AC_OUT[E_AC<br/>空調一次エネルギー消費量<br/>MJ/年]
        DETAIL[内訳<br/>• 空調機<br/>• ポンプ<br/>• 熱源<br/>• 補機]
    end

    XML_IN --> STEP2
    WEATHER_IN --> STEP1
    CONST_IN --> STEP3

    STEP1 --> STEP3
    STEP2 --> STEP3

    STEP3 --> L1
    STEP3 --> L2
    STEP3 --> L3
    STEP3 --> L4
    STEP3 --> L5

    L1 --> STEP4
    L2 --> STEP4
    L3 --> STEP4
    L4 --> STEP4
    L5 --> STEP4

    STEP4 --> STEP5
    STEP5 --> STEP6
    STEP6 --> STEP7
    STEP7 --> STEP8
    STEP8 --> STEP9

    STEP9 --> E_AC_OUT
    STEP9 --> DETAIL

    style XML_IN fill:#e1ffe1
    style WEATHER_IN fill:#ffffcc
    style CONST_IN fill:#ffffcc
    style STEP1 fill:#e1f5ff
    style STEP2 fill:#e1f5ff
    style STEP3 fill:#ffe1e1
    style STEP4 fill:#ffe1e1
    style STEP5 fill:#ffe1e1
    style STEP6 fill:#ffd700
    style STEP7 fill:#ffd700
    style STEP8 fill:#ffd700
    style STEP9 fill:#90ee90
    style E_AC_OUT fill:#90ee90
```

### 3.1 負荷率帯分類の詳細

```mermaid
graph LR
    subgraph "各時刻の負荷"
        Q[実負荷 Q_t]
        Q_RATED[定格能力 Q_rated]
    end

    subgraph "負荷率計算"
        ALPHA[負荷率 α = Q_t / Q_rated]
    end

    subgraph "11区分への分類"
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

    subgraph "時間数集計"
        T[T_hr,n<br/>各区分の時間数]
    end

    Q --> ALPHA
    Q_RATED --> ALPHA

    ALPHA --> B0
    ALPHA --> B1
    ALPHA --> B2
    ALPHA --> B3
    ALPHA --> B4
    ALPHA --> B5
    ALPHA --> B6
    ALPHA --> B7
    ALPHA --> B8
    ALPHA --> B9
    ALPHA --> B10

    B0 --> T
    B1 --> T
    B2 --> T
    B3 --> T
    B4 --> T
    B5 --> T
    B6 --> T
    B7 --> T
    B8 --> T
    B9 --> T
    B10 --> T

    style ALPHA fill:#ffe1e1
    style T fill:#90ee90
```

## 4. 計算結果の統合フロー

```mermaid
flowchart TD
    subgraph "各設備の年間エネルギー消費量"
        E_AC[E_AC: 空調<br/>空調機+ポンプ+熱源]
        E_V[E_V: 換気<br/>送風機]
        E_L[E_L: 照明<br/>照明器具]
        E_HW[E_HW: 給湯<br/>給湯機器+配管損失]
        E_EV[E_EV: 昇降機<br/>エレベータ]
        E_M[E_M: その他<br/>OA機器等]
    end

    subgraph "削減効果"
        E_PV[E_PV: 太陽光発電<br/>自己消費分]
        E_CGS[E_CGS: コージェネ<br/>発電+排熱利用]
    end

    subgraph "統合計算（第1章）"
        SUM[E_T = E_AC + E_V + E_L + E_HW + E_EV + E_M - E_PV - E_CGS]
        UNIT[MJ/年 → GJ/年<br/>× 10^-3]
        ROUND[小数第2位切り上げ]
    end

    subgraph "基準値計算（第10章）"
        ST_SUM[E_ST = E_SAC + E_SV + E_SL + E_SEV + E_SHW + E_M]
        ST_COEF[室用途別×地域別<br/>標準係数適用]
        ST_UNIT[MJ/年 → GJ/年<br/>× 10^-3]
        ST_ROUND[小数第2位切り上げ]
    end

    subgraph "最終結果"
        E_T_FINAL[E_T: 設計一次エネルギー消費量<br/>GJ/年]
        E_ST_FINAL[E_ST: 基準一次エネルギー消費量<br/>GJ/年]
        RATIO[BEI = E_T / E_ST<br/>建築物省エネルギー性能表示指標]
        JUDGE{BEI ≦ 1.0?}
        PASS[基準適合]
        FAIL[基準不適合]
    end

    E_AC --> SUM
    E_V --> SUM
    E_L --> SUM
    E_HW --> SUM
    E_EV --> SUM
    E_M --> SUM
    E_PV --> SUM
    E_CGS --> SUM

    SUM --> UNIT
    UNIT --> ROUND
    ROUND --> E_T_FINAL

    ST_COEF --> ST_SUM
    ST_SUM --> ST_UNIT
    ST_UNIT --> ST_ROUND
    ST_ROUND --> E_ST_FINAL

    E_T_FINAL --> RATIO
    E_ST_FINAL --> RATIO

    RATIO --> JUDGE
    JUDGE -->|Yes| PASS
    JUDGE -->|No| FAIL

    style SUM fill:#ffd700
    style ST_SUM fill:#ffd700
    style E_T_FINAL fill:#90ee90
    style E_ST_FINAL fill:#90ee90
    style RATIO fill:#ff6347
    style PASS fill:#32cd32
    style FAIL fill:#dc143c
```

## 5. データの依存関係

```mermaid
graph TD
    subgraph "基礎データ"
        F0[様式0: 基本情報<br/>地域区分・階数]
        F1[様式1: 室仕様<br/>室面積・用途]
    end

    subgraph "設備データ"
        F2[様式2: 空調]
        F3[様式3: 換気]
        F4[様式4: 照明]
        F5[様式5: 給湯]
        F6[様式6: 昇降機]
        F7[様式7: 発電・コージェネ]
    end

    subgraph "計算への依存"
        CALC_AC[空調計算]
        CALC_V[換気計算]
        CALC_L[照明計算]
        CALC_HW[給湯計算]
        CALC_EV[昇降機計算]
        CALC_PV[太陽光計算]
        CALC_CGS[コージェネ計算]
        CALC_M[その他計算]
        CALC_ST[基準値計算]
    end

    F0 -.地域区分.-> CALC_AC
    F0 -.地域区分.-> CALC_HW
    F0 -.地域区分.-> CALC_PV
    F0 -.地域区分.-> CALC_ST

    F1 -.室定義.-> F2
    F1 -.室定義.-> F3
    F1 -.室定義.-> F4
    F1 -.室定義.-> F5

    F1 -.室面積.-> CALC_M
    F1 -.室面積.-> CALC_ST

    F2 --> CALC_AC
    F3 --> CALC_V
    F4 --> CALC_L
    F5 --> CALC_HW
    F6 --> CALC_EV
    F7 --> CALC_PV
    F7 --> CALC_CGS

    CALC_AC -.排熱負荷.-> CALC_CGS
    CALC_HW -.給湯負荷.-> CALC_CGS

    style F0 fill:#ffd700
    style F1 fill:#ffd700
    style CALC_AC fill:#e1f5ff
    style CALC_V fill:#e1f5ff
    style CALC_L fill:#e1f5ff
    style CALC_HW fill:#e1f5ff
    style CALC_EV fill:#e1f5ff
    style CALC_PV fill:#90ee90
    style CALC_CGS fill:#90ee90
    style CALC_M fill:#e1f5ff
    style CALC_ST fill:#ffd700
```

## 6. データ量の推定

| データ種別 | データ量 | 説明 |
|----------|---------|------|
| CSV入力シート | 約1～10KB/様式 | 1建物あたり |
| XML統合データ | 約50～500KB | 1建物あたり（規模による） |
| 気象データ | 8760時間×5変数×8地域 | 約350KB（すべての地域） |
| 定数・係数データ | 数MB | 建材、機器特性等 |
| 計算中間データ | 数MB～数十MB | 8760時間×室数の負荷データ等 |
| 計算結果 | 数KB | 統合値と内訳 |

## 7. データの時間粒度

| 計算対象 | 時間粒度 | データ点数 |
|---------|---------|----------|
| 気象条件 | 1時間 | 8760点/年 |
| 室負荷計算 | 1時間 | 8760点/年/室 |
| 空調機負荷 | 1時間 → 負荷率帯 | 8760点 → 11区分 |
| 給湯負荷 | 1日 | 365点/年 |
| 照明点灯 | 年間積算 | 1点/年/室 |
| 昇降機運転 | 年間積算 | 1点/年/機器 |
| 最終結果 | 年間積算 | 1点/年 |

## 8. データのバリデーション

```mermaid
flowchart TD
    INPUT[CSV入力データ]

    subgraph "バリデーション"
        V1{必須項目<br/>入力済み?}
        V2{データ型<br/>正しい?}
        V3{値の範囲<br/>妥当?}
        V4{外部キー<br/>参照先存在?}
        V5{論理整合性<br/>OK?}
    end

    ERROR[エラーメッセージ<br/>入力箇所指摘]
    OK[バリデーション<br/>合格]
    CONVERT[XML変換実行]

    INPUT --> V1
    V1 -->|No| ERROR
    V1 -->|Yes| V2
    V2 -->|No| ERROR
    V2 -->|Yes| V3
    V3 -->|No| ERROR
    V3 -->|Yes| V4
    V4 -->|No| ERROR
    V4 -->|Yes| V5
    V5 -->|No| ERROR
    V5 -->|Yes| OK
    OK --> CONVERT

    style ERROR fill:#dc143c
    style OK fill:#32cd32
    style CONVERT fill:#90ee90
```

### バリデーション項目の例

**必須項目チェック:**
- 建物名（様式0）
- 地域区分（様式0）
- 室名（様式1）
- 室面積（様式1）

**データ型チェック:**
- 数値項目に数値が入力されているか
- 日付項目が正しい形式か
- 列挙値が定義済みの値か

**値の範囲チェック:**
- 面積 > 0
- 階高 > 天井高
- 効率 0～1（または0～100%）
- 地域区分 1～8

**外部キー参照チェック:**
- 空調ゾーンに含まれる室名が様式1に存在するか
- 給湯機器名称が様式5-2に定義されているか
- 熱源群名称が様式2-5に存在するか

**論理整合性チェック:**
- 冷房設定温度 < 暖房設定温度
- 窓面積の合計 ≦ 外皮面積
- 定格能力 > 0

## 9. まとめ

Webproシステムのデータフローは以下の特徴を持ちます：

1. **多段階変換**: CSV → XML → 計算結果 → 評価レポート
2. **設備別並列処理**: 7つの設備計算が並列実行可能
3. **時間粒度の多様性**: 時間単位・日単位・年単位が混在
4. **厳密なバリデーション**: 入力データの整合性を多段階でチェック
5. **透明性**: すべての変換ルールと計算式が公開
6. **拡張性**: 新規設備カテゴリの追加が容易
