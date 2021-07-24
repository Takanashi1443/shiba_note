# InputEventSystem

[トップへ](./../../index.md)

## InputEventSystemの仕組み

入力には、

- キーボード
- マウス
- コントローラー(PC用)
- コントローラー(商業機)
- タッチパネル

など、媒体によって様々なものがある。

ShibaEngineでは、これらにある程度対応しつつも、各ShibaAssetが入力システムの制約を受けないよう、
入力を一旦イベントに変換して各コンポーネントに入力を検知させる方法を取る。

例えば、

- キーボードの決定キー
- マウスの左クリック
- コントローラーの決定ボタン
- 画面タッチによりゲームが進行する場合の、画面タッチ
- UIとして表示された決定ボタンのタッチ

は、いずれも「決定」「進む」の意思表示と捉えられる。

ShibaEngineでは、InputManagerというコンポーネントが上記の入力を検知すると
「決定」「進む」を表す「INPUT_CONFIRM」というイベントを発行するようにする、といった仕組みを予め準備している。

InputSystemは、大きく分けて

- 入力を検知し、入力イベントに変換するInputManager
- 発行された入力イベントを検知するInputListener

からなっている。

この方法では表現できない入力として、

- アナログスティックなど、数値情報を含む入力
- タッチバッドなど、位置情報を含む入力

などが考えられるが、InputEventSystemはこれらは扱わない。

関連するスクリプトは、

```

Plugins/ShibaEngine/Core/Scripts/InputEventSystem

```

フォルダ内に入っている。

## InputEventsの種類

InputEventSystemを使用する場合、まずシーンで扱う入力に必要十分な「InputEvents」の種類を選択する。

CommandInputEvents
: RPGなどのコマンド入力を行うゲームを想定したInputEventsで、
: 上下左右キーの入力、決定/キャンセルの入力、Alt1/ALt2を押す/離すという入力に対応する。

## システムを構成するコンポーネント群の説明

ここでは、CommandInputEventsを使用した場合のシステムを構成するコンポーネント群について説明する。

（現在、CommandInputEvents以外のInputEventsはコンポーネントを作っていない）

スクリプトは

```

Plugins/ShibaEngine/Core/Scripts/InputEventSystem

```

内にあり、ここを(ルート)と表記する。

### CommandInputEvents

扱うイベントを表現する列挙体であるCommandInputEventsは

```

(ルート)/InputEventTypes

```

フォルダ内にファイルがある。

### CommandInputManagerBase

InputManager。

入力の解釈方法は派生クラスで定義されている。

SceneManagerと似た機能を持つが、SceneEventは自身もイベントを検知するのに対し、自身はリスナーとしての機能を持たない。

場所は

```

(ルート)/Components/CommandInput

```

#### StylizedKeyboardCommandInputManager

キーボード入力を解釈してCommandInputを発行するInputManager。

場所は

```

(ルート)/Components/SceneManager

```

で、CommandInputManagerBaseと違うフォルダ内にあるが、これは、InputManagerBaseは各InputTypeに応じて決まったものがあるが、
InputManagerは入力媒体によって多数存在しうるため。

### CommandInputTransmitter / CommandInputReciever

シーンイベントと同様、シーンイベントを発行・購読する際にはこれらのコンポーネントがInjectされる。

### CommandInputInstaller

上記のコンポーネントをInjectするためのMonoInstaller。

シーンのSceneContext内にセットする。

