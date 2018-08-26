# 第10章: Segregated Witness
トランザクションの署名をトランザクションの入力部からwitnessと呼ばれる別のデータ領域に移すSegregated Witnessと呼ばれる

## トランザクションmalleabilityの問題
ビットコインのトランザクションを構成するデータとして重要なのは
- inputとなるUTXOへの参照セット
- 新しいoutputのセット

トランザクションの識別子であるTXIDはscript_sigも含めた上記データから生成されたハッシュ値である  
→ script_sig部のデータが改竄されるとTXIDも別の値になる  

署名検証に影響を与えないようにscript_sigを変更すればTXIDの異なるトランザクションを作れる  

署名スクリプトのmalleability問題を根本的に解決するのがSegregated Witness
- Segwitではトランザクションのインプットから署名を分離し、署名を別のデータ領域に移動することでこの問題を解決する








































