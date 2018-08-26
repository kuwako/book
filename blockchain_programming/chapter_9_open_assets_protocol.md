# 第9章: OpenAssetsProtocol
ビットコインのブロックチェーンの「改ざんが困難」「誰もが参照可能な共有台帳」という特性を活かして応用する

### OpenAssetsProtocolとは
2017年5月現在
- Python
- Ruby
- C# 
- Objective-C

で開発可能。  
OpenAssetsProtocolはブロックチェーン城でビットコイン以外のアセットを発行・取引するために以下の機能を定義している
- MarketOutput
    - トランザクションがビットコインではなくアセットを送付しているトランザクションであることを示すアウトプット
- AssetQuantity
    - 取引でいくつのアセットが取引されているのか示すアセットの量
- AssetID
    - ブロックチェーン城で取引されるアセットを一意に識別するためのID
- OpenAssetsAddress
    - ビットコインアドレスと1対1に対応するアセットを取引する際に使用するアドレス
- AssetDefinitionProtocol
    - アセットに関するメタ情報とアセットを紐づけるためのプロトコル

## Marker Output
OpenAssetsProtocolのトランザクションには必ずMarketOutput(特殊なアウトプット)がある






































