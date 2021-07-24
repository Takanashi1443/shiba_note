# フォルダ構成

[トップへ](./../../index.md)

## 前提知識の概説

### アセンブリ分割

コンパイルの単位を分割することをアセンブリ分割という。

アセンブリ分割はフォルダ内にアセンブリ定義ファイル（拡張子:.asmdef）を置くことで行われる。

アセンブリ定義ファイル内に、当該アセンブリが他のどのアセンブリに依存しているかなどの情報も保持する。

アセンブリ分割を行うことで、ファイルを更新した時にそれと関係のないファイルまで再コンパイルされるのを防ぎ、
コンパイルが高速化される。

### Pluginフォルダ

Pluginフォルダ内は、Pluginフォルダ外よりも先にコンパイルされる。アセンブリ分割に似ている。

Pluginフォルダ内からPluginフォルダ外を参照できない。

変更されることが少ないものはPluginフォルダ内に入れる。

## フォルダ構成

```

Assets
├ ShibaEngine
｜  ├ HogeAssetA
｜  ｜　├ Scripts 
｜  ｜　├ CopySource
｜  ｜  ｜　└ ShibaEngine.HogeAssetA.CopySource.asmdef 
｜  ｜　└ ShibaEngine.HogeAssetA.asmdef 
｜　└ (以下略)
｜
├ Plugins
｜　├ UniRx
｜  ｜　┠ Scripts 
｜  ｜  ｜　└ UniRx.asmdef 
｜  ｜　└ Examples
｜  ｜
｜　├ Zenject
｜  ｜　└ zenject.asmdef 
｜  ｜
｜　├ ShibaEngine
｜　｜　├ Core
｜  ｜  ｜　├ Scripts
｜  ｜  ｜  ｜　├ EventSystem 
｜  ｜  ｜  ｜　├ InputSystem 
｜  ｜  ｜  ｜　└ (以下略)
｜  ｜  ｜  ｜
｜  ｜  ｜　├ CopySource
｜  ｜  ｜  ｜　└ ShibaEngine.Core.CopySource.asmdef 
｜  ｜  ｜　└ ShibaEngine.Core.asmdef
｜  ｜  ｜
｜　｜　├ Modules
｜  ｜  ｜　├ Scripts
｜  ｜  ｜  ｜　├ OutputCurve
｜  ｜  ｜  ｜　└ (以下略)
｜  ｜  ｜  ｜
｜  ｜  ｜　├ CopySource
｜  ｜  ｜  ｜　└ ShibaEngine.Modules.CopySource.asmdef 
｜  ｜  ｜　└ ShibaEngine.Modules.asmdef
｜  ｜  ｜
｜　｜　└ Utils
｜  ｜  　  └ ShibaEngine.Utils.asmdef
｜  ｜
｜　└ (以下略)
｜
└ （以下略)

```

## 解説

ShibaEngineは、

- 中核のスクリプトであるCore
- 汎用的で、複数のShibaAssetから参照されるModules
- 主にエディタ拡張や拡張メソッドを定義するUtils
- 個別の機能を提供するShibaAssets（複数）

からなる。

このうち、ShibaAssets以外は、先だってコンパイルされるPluginsフォルダに入れてあり、
ShibaAssetsのみPluginsの外にある。
（ShibaAssets、Plugins/ShibaAssetsはProjectビューでFavoritesに入れておくとよい）

スクリプトの使用例であり、独自のスクリプトを記述する際にコピペ元とすることを想定したアセット群を
CopySourceと呼ぶ。
CopySourceは、関連するスクリプトと同じフォルダ内にCopySourceというフォルダを作って
そこに定義するが、アセンブリ分割はCopySourceとそれ以外で分ける。
