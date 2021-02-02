# mem

# powershell
- 娑婆のRSAをやりたい https://qiita.com/rawr/items/9492e7fc93be7ca99ac1
- RSA鍵の書式 https://qiita.com/hotpepsi/items/128f3a660cee8b5467c6
- .NETのAPIについて https://qiita.com/acple@github/items/e80bef939583fc2b0e5e
## openssl 
- Expecting: TRUSTED CERTIFICATE エラー
  https://humo-life.net/memo/doku.php?id=%E3%82%B5%E3%83%BC%E3%83%90:ssl%E8%A8%BC%E6%98%8E%E6%9B%B8:no_start_line_pem_lib.c_707_expecting_trusted_certificate
  ファイルが UTF-8、BOM有、CR+LF になっている？UTF-8、BOM無、LF でファイルを保存してみる。
- 読み込むのは暗号化された"バイナリ" .pcapだったらwiresharkでas bits表示して、パケットバイト列を生bit列としてエクスポートする。
  https://qiita.com/Shinya-Yamaguchi/items/becd042f641362c432bc | 
  https://qiita.com/toshihirock/items/acbf9800f7e784118e46 | 
- ヘッダ等の文字列の問題？ https://sabakan-haruka.hatenadiary.org/entry/20080430/1209536052 |
- 暗号文の破損？ https://jpcloud.net/q/nmpcaxam | 
- 手動で復号 https://lowleveldesign.org/2016/03/09/manually-decrypting-https-request/ | 

# wireshark
- Encrypted Handshake Message や TLSで送られている Application Data などはRSAで暗号化されているとする。
- TLSの"Encrypted ..."をバイナリとしてエクスポートしても、RSA復号はうまくいかないかもしれない。
- wiresharkで復号してもらったほうが早い。
- 編集 > 設定 > "RSA keys" > .pemを追加する。 > 右クリック > ...としてデコード > OK

# SSL/TLS
- SSL/TLS https://qiita.com/Brutus/items/1015cc01d2e1eb82a526 |
- Encrypted...部分のバイナリをopensslでデコードできなかった。なぜか？バイナリを取ってくる場所か方法を間違えた？
- TLSのRFCの和訳 https://www.ipa.go.jp/security/rfc/RFC5246-06JA.html#062

# SSLのこと
- 秘密鍵は2種類ある
  - 1:サーバの持つ秘密鍵。公開鍵の暗号文を復号する。
  - 2:サーバとクライアントA間で共有する秘密鍵。Handshake後にクライアントAが生成。
- 2:をクライアントAが生成し、公開鍵で暗号化してサーバに渡す。
- 手に入った秘密鍵が1:なら Client Key Exchange とかを探して それを復号する。

# 共通鍵の導出
- preMaster secret > master secret > MAC secret + "session key"
- 7.4.7.1 鍵交換と認証方式として RSA が使用されているならば、クライアントは 48 バイトの premaster_secret を生成し、サーバー証明書から取得した公開鍵、または ServerKeyExchange メッセージにて提供される一時的 RSA 鍵を使用して暗号化する。その結果を EncryptedPreMasterSecret メッセージに含め送信する。この構造は ClientKeyExchange メッセージの一種であり、これ自体は独立したメッセージというわけではない。
- すべての鍵交換方式において、pre_master_secret を master_secret へ変換するのに、同一のアルゴリズムが使用される。master_secretが計算されると、 pre_master_secret はメモリ上から削除されるべきである。

master_secret = PRF(pre_master_secret, "master secret",
ClientHello.random + ServerHello.random)
[0..47];

master_secret の長さは、常に 48 バイトである。
- 8.1 暗号化preMaster secret の構造

struct {

ProtocolVersion client_version;
opaque random[46];

} PreMasterSecret;

client_version
クライアントがサポートしている、最新のバージョン。これは、バージョンロールバック攻撃を検出するのに使用される。この premaster_secret を受信すると、サーバーは、この値が ClientHello メッセージで受信した値と同じであるかをチェックすべきである。
　
random
安全に生成された 46バイトの乱数。
struct {

public-key-encrypted PreMasterSecret pre_master_secret;

} EncryptedPreMasterSecret;

- 
