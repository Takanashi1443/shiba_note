# 独自のイベントを定義したシーン

[トップへ](./../../index.md)

## シーンイベントの実装方法

### イベントの種類を列挙体として定義する

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

### SceneManagerを派生クラスとして定義

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

### Zenjectのシーン用MonoInstallerの定義


### イベントリスナの動作

イベントマネージャは、外部に対してIObservable\<SceneEvents\>として公開する。


### イベントコーラーの動作


## GameControllerモジュール内のソースコードの解説




