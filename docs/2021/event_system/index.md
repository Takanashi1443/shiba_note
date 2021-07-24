# イベントシステム

[トップへ](./../../index.md)

## イベントシステムとは

例えばアクションゲームで、プレイヤーが倒れた時、特定ポイントを通過した時など、
特定のタイミングで複数のものに影響を与えたい場合がある。
これは、例えば、ゲームマネージャがプレイヤーの状態、全てのポイントの状態を把握し、その状態の変化に応じて
影響を与えるもの全てに対して処理を行うという方法でも実装できるが、
影響を与える対象が増えるたびにゲームマネージャを書き換えなければならず、不便である。
そこで、「イベント」という仕組みを使う。

各コンポーネントは、ゲームマネージャに管理されていなくても「イベント」を発行することができ、
「イベント」が発行されると、それに反応する各コンポーネントたちが勝手に反応する。

イベントは様々な種類を定義できるが、最も基本的なものはシーンの進行に関する「シーンイベント」である。

「ステージ開始」や「ステージクリア」といった、よくあるイベントを発行対象とする。

シーンイベントでは、パラメータを渡すことは出来ないものとする。
例えば、クイズにおいて、「正解した」というイベントは作れるが、「3番を選んで正解した」（3は他の数にもなりうる）のような、
パラメータを伴うイベントは作れない。

ここではシーンイベント、特にShibaEngineのデフォルトで用意している「StylizedSceneEvents」の詳細について説明する。

## シーンイベント

### はじめに

シーンイベントで使用するスクリプト群は、Plugins/ShibaEngine/Core/Script/EventSystem 内にある。

#### 5種類のコンポーネント

イベントシステムには、以下の5種類のコンポーネントが関連する。

- シーンマネージャ (SceneManager)
- コーラー (SceneEventCaller)
- リスナー (SceneEventListener)
- トランスミッター (SceneEventTransmitter)
- レシーバ (SceneEventReceiver)

の5つが存在する。

また、使用するシーンイベントの一覧を列挙型で定義するクラスSceneEventsがある。

上記5種類のコンポーネントは全て、列挙型であるSceneEvents（シーンで発生するイベント）を持つジェネリックとして定義される。

(後で図にしたい)
上記の5つのうち、コーラーは、シーンイベントを発行するもの全般、リスナーは、シーンイベントを受けて何かするもの全般を指す。

一方、トランスミッターはコーラーから利用され、シーンマネージャを参照してイベントを発行するだけの機能を持つコンポーネント、
レシーバはシーンマネージャを参照してイベントの発行を受信しリスナーに伝えるだけの機能を持つコンポーネントである。

#### 各コンポーネントの動作の説明

##### シーンマネージャ

シーンマネージャは、購読対象となるストリームの実体（Subject\<SceneEvents\>）を持ち、IObservable\<SceneEvents\>を外部に公開する。
トランスミッターから参照され、イベントを引数としてイベント実行を試みる関数CallEventを実行される。
シーンマネージャは、イベント実行を許可する場合（ValidEvent関数による検証）、CallEventで指定された値のストリームを発行する。

シーンマネージャは、トランスミッターがイベント実行を要求しても棄却する権限を持つ。

##### トランスミッター

トランスミッターは、コーラーと同じGameObjectにアタッチされ、参照される。

トランスミッターはシーンマネージャを参照し、コーラーからCallEvent関数を実行された時、シーンマネージャのInvokeEvent関数を実行する。

コンポーネントは原則、トランスミッター経由以外でシーンマネージャのInvokeEvent関数を実行してはならない。

##### レシーバ

レシーバは、リスナーと同じGameObjectにアタッチされ、相互に参照する。

レシーバはシーンマネージャを参照し、IObservable\<SceneEvents\>を購読する。シーンマネージャからイベントが発行された時、
リスナーのOnEventCalled関数を実行することでそれをリスナーに伝える。

コンポーネントは原則、レシーバ経由以外でシーンマネージャのストリームを購読してはならない。

##### コーラーとリスナー

コーラーはトランスミッターを参照するもの、リスナーはレシーバを参照するもの全般を指す。

### StylizedSceneEvents

ShibaEngineはデフォルトで、割と色々なシーンに使用できるシーンイベントの一覧を定義したクラスであるStylizedSceneEventsを定義している。

Plugins/ShibaEngine/Core/Script/EventSystem/StylizedSceneEvents.cs

以下に、StylizedSceneEventsで定義されているイベントの一覧を示す。
これらのイベントを使ってシーンを回すことが出来るようであれば、自作のシーンを作る場合もこのStylizedSceneEventsを使うとよい。

#### 基本のイベント

| 列挙子 | 内容 | 備考 |
| :---: | :--- | :--- |
| UNDEFINED | 未定義。無意味なイベントを意味させたい時に使用。 |  |
| OPEN_START | フェードインなどの遷移でシーンを開始する際の遷移処理の開始。<br>SceneTransitterはこのイベントで駆動し、OPEN_FINISHイベントを発行する。 |  |
| OPEN_FINISH | 遷移処理の終了。 |  |
| CLOSE_START | シーン終了時の遷移処理の開始。<br>SceneTransitterはこのイベントで駆動し、CLOSE_FINISHイベントを発行する。 |  |
| CLOSE_FINISH | 遷移処理の終了。 |  |
| PLAY_START | シーンのメイン部分の開始。 |  |
| PLAY_FINISH | シーンのメイン部分の終了。 |  |
| RESULT_START | シーンの結果の表示の開始。 |  |
| RESULT_FINISH | シーンの結果の表示の終了。 |  |
| PAUSE_ON | 一時停止のON。 |  |
| PAUSE_OFF | 一時停止のOFF。 |  |
| MENU_ON | メニュー画面を開く。 |  |
| MENU_OFF | メニュー画面を閉じる。 |  |
| ROUND_START | ターン開始。 |  |
| ROUND_END | ターン終了。 |  |

#### 入力

単純なシーンでは、プレイヤーの入力を直接イベントの発行に結びつけた方が作りやすい場合がある。

そのような場合に使用する。ある程度複雑なシーンではInputManagerなどを使用する方が良い。

また、どのキーを押した時にどのイベントを発行するかは自前で準備する必要がある。

| 列挙子 | 内容 | 備考 |
| :---: | :--- | :--- |
| INPUT_PROCEED | 進むや決定を表す入力。 |  |
| INPUT_BACK |  | 戻るやキャンセルを表す入力。 |
| INPUT_COMMAND1 | その他の入力。内容は自分で定義する。以下同様。 |  |
| INPUT_COMMAND2 |  |  |
| INPUT_COMMAND3 |  |  |
| INPUT_COMMAND4 |  |  |

#### 成功/失敗関係

| 列挙子 | 内容 | 備考 |
| :---: | :--- | :--- |
| EVENT_SUCCESS | プレイの成功。 |  |
| EVENT_FAIL | プレイの失敗。 |  |
| PLAY_SUCCESS_FINISH | プレイが成功のもとに終了。 |  |
| PLAY_FAIL_FINISH | プレイが失敗のもとに終了。 |  |
| RESULT_SUCCESS_START | 成功のもとに終了した場合のリザルト画面開始。 |  |
| RESULT_FAIL_START | 失敗のもとに終了した場合のリザルト画面開始。 |  |
| RESULT_SUCCESS_FINISH | 成功のもとに終了した場合のリザルト画面終了。 |  |
| RESULT_FAIL_FINISH | 成功のもとに終了した場合のリザルト画面終了。 |  |

#### その他

| 列挙子 | 内容 | 備考 |
| :---: | :--- | :--- |
| EVENT1 | 汎用イベント。内容は自分で定義する。以下同じ。 |  |
| EVENT2 |  |  |
| EVENT3 |  |  |
| EVENT4 |  |  |
| MODE1_ON | モード1のON。内容は自分で定義する。以下同じ。 |  |
| MODE1_OFF | モード1のOFF。 |  |
| MODE2_ON |  |  |
| MODE2_OFF |  |  |
| MODE3_ON |  |  |
| MODE3_OFF |  |  |
| MODE4_ON |  |  |
| MODE4_OFF |  |  |

### StylizedSceneEventsをシーンイベントとして使用する場合のシーンの作り方


### StylizedSceneEventsを実際のシーンで使用する例

#### 基本的に一方通行で、分岐もなく、成功や失敗の判定もないシーン（会話シーンなど）

| 発行するタイミング | 発行するイベント | イベント発行者 | 備考 |
| :--- | :--- | :--- | :---|
| シーン開始演出開始。 | OPEN_START | SceneManager |  |
| シーン開始演出終了。 | OPEN_FINISH | SceneTransitter |  |
| プレイ開始。 | PLAY_START | SceneManager | 上に連続 |
| シーンを先に進めるために使用。 | INPUT_PROCEED | InputManager | 適宜 |
| プレイ終了。 | PLAY_FINISH | 独自コンポーネント |  |
| シーン終了演出開始。 | CLOSE_START | SceneManager | 上に連続 |
| シーン終了演出終了。 | CLOSE_FINISH | SceneTransitter |

#### プレイを行い、成功または失敗の結果になるシーン

（共通）

| 発行するタイミング | 発行するイベント | イベント発行者 | 備考 |
| :--- | :--- | :--- | :---|
| シーン開始演出開始。 | OPEN_START | SceneManager |  |
| シーン開始演出終了。 | OPEN_FINISH | SceneTransitter |  |
| プレイ開始。 | PLAY_START | SceneManager | 上に連続 |

（成功した場合）

| 発行するタイミング | 発行するイベント | イベント発行者 | 備考 |
| :--- | :--- | :--- | :---|
| 成功フラグ。 | EVENT_SUCCESS | 独自コンポーネント |  |
| プレイ（成功）終了。 | PLAY_SUCCESS_FINISH | SceneManager | 上を受けて処理後、すぐ |
| リザルト（成功）の表示開始。 | RESULT_SUCCESS_START | 独自コンポーネント |  |
| リザルト（成功）の表示終了。 | RESULT_SUCCESS_FINISH | 独自コンポーネント |  |

（失敗した場合）

| 発行するタイミング | 発行するイベント | イベント発行者 | 備考 |
| :--- | :--- | :--- | :---|
| 失敗フラグ。 | EVENT_FAIL | 独自コンポーネント |  |
| プレイ（失敗）終了。 | PLAY_FAIL_FINISH | SceneManager | 上を受けて処理後、すぐ |
| リザルト（失敗）の表示開始。 | RESULT_FAIL_START | 独自コンポーネント |  |
| リザルト（失敗）の表示終了。 | RESULT_FAIL_FINISH | 独自コンポーネント |  |

（共通）

| 発行するタイミング | 発行するイベント | イベント発行者 | 備考 |
| :--- | :--- | :--- | :---|
| シーン終了演出開始。 | CLOSE_START | SceneManager | 上に連続 |
| シーン終了演出終了。 | CLOSE_FINISH | SceneTransitter |

### シーンイベントの実装方法

#### イベントの種類を列挙体として定義する

シーンごとのイベントは、イベントの種類を列挙した専用の列挙体および、
その列挙体を取り扱う専用のReactivePropertyを定義する。
例えば、あるシーン（シーン名：Stage）に、

- READY … シーンの準備完了。これを実行後操作可能になる
- DEATH … プレイヤーの死亡。
- CLEAR … プレイヤーのクリア。

の3つのイベントを用意するとする。
この時、
Assets/HogeProject/Scripts/FugaSceneフォルダ内に、
SceneEvents.csスクリプトを作り、以下のように記述する。

```

namespace (Project Name).StageScene
{
    /// <summary>
    /// イベントの列挙。
    /// </summary>
    public enum SceneEvents
    {
        READY,
        DEATH,
        CLEAR,
    }
}

```

#### SceneManagerを派生クラスとして定義

GameControllerに定義されているSceneManagerBase\<SceneEvents\>またはStandardSceneManager\<SceneEvents\>を派生させ、
シーン固有のSceneManagerクラスを定義する。
クラス名はSceneManagerで良い。

イベントマネージャはイベント発行用に Subject\<SceneEvents\> を持つ。

```
using UniRx;

namespace (Project Name).StageScene
{
    public class SceneEventManager : MonoBehaviour, ISceneEventManager
    {
        /// <summary>
        /// enum値で発行されるイベントのストリーム。
        /// </summary>
        protected Subject<SceneEvents> m_sceneEvent = new Subject<SceneEvents>();

        (中略)
    }
}

```

#### Zenjectのシーン用MonoInstallerの定義


#### イベントリスナの動作

イベントマネージャは、外部に対してIObservable\<SceneEvents\>として公開する。


#### イベントコーラーの動作


### GameControllerモジュール内のソースコードの解説




