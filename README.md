# SistemYoneticiligi_Zabbix

# Zabbix Kurulum Komutları

## 1.	Root kullanıcı olun:
### Root ayrıcalıklarıyla yeni bir kabuk oturumu başlatın
$ sudo -s

## 2.	Zabbix deposunu kurun:

 wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
 dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
 apt update

## 3.	Zabbix sunucusunu, arayüzünü ve aracısını kurun:

 apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent

## 4.	İlk veritabanını oluşturun: 
### MySQL'e root olarak bağlanın (kurulum sırasında belirlediğiniz parolayı girmeniz istenecektir)

mysql -uroot -p
password
mysql> create database zabbix character set utf8mb4 collate utf8mb4_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> set global log_bin_trust_function_creators = 1;
mysql> quit;

### Şimdi Zabbix şemasını ve verilerini içe aktarın

 zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix

### Veritabanı şemasını içe aktardıktan sonra log_bin_trust_function_creators seçeneğini devre dışı bırakın

mysql -uroot -p
password
mysql> set global log_bin_trust_function_creators = 0;
mysql> quit;

## 5.	Zabbix sunucusu için veritabanını yapılandırın:
Edit file /etc/zabbix/zabbix_server.conf

DBPassword=password

## 6.	Zabbix sunucusu ve aracısı işlemlerini başlatın: 
### Zabbix sunucusunu, aracısını ve Apache'yi yeniden başlatın ve açılışta otomatik başlamalarını sağlayın

 systemctl restart zabbix-server zabbix-agent apache2
 systemctl enable zabbix-server zabbix-agent apache2

## 7.	Zabbix arayüzünü yapılandırın: 
Bir web tarayıcısı açın ve Zabbix sunucunuzun IP adresini veya alan adını girerek Zabbix arayüzüne gidin: http://host/zabbix[1]


# Webhook Script Kodu
try {
    var params = JSON.parse(value);
    
    // Gerekli parametrelerin kontrolü
    if (!params.Token) {
        throw 'Bot token "Token" tanımlanmamış.';
    }
    if (!params.To) {
        throw 'Alıcı "To" (chat_id) tanımlanmamış.';
    }
    if (!params.Message) {
        throw 'Parametre "Message" tanımlanmamış.';
    }
    
    // Subject (Konu) genellikle sağlanır ama mesajın bir parçası olarak kullanılır.
    // ParseMode isteğe bağlıdır; Telegram belirtilmemişse veya geçersizse varsayılanı kullanır.
    
    var request = new HttpRequest();
    
    // İsteğe bağlı: Proxy desteği (Eğer Zabbix arayüzünde HTTPProxy parametresi eklerseniz)
    if (typeof params.HTTPProxy === 'string' && params.HTTPProxy.trim() !== '') {
        request.setProxy(params.HTTPProxy);
        Zabbix.Log(4, '[Telegram Webhook] Proxy kullanılıyor: ' + params.HTTPProxy);
    }
    
    request.addHeader('Content-Type: application/json');
    
    var message_text = params.Message;
    
    // Eğer Subject (Konu) parametresi doluysa, mesajın başına ekle
    if (typeof params.Subject === 'string' && params.Subject.trim() !== '') {
        message_text = params.Subject + '\n' + params.Message;
    }
    
    var body = {
        chat_id: params.To,
        text: message_text
    };
    
    // ParseMode parametresini ayarla (Markdown, HTML, MarkdownV2)
    if (typeof params.ParseMode === 'string' && ['Markdown', 'HTML', 'MarkdownV2'].indexOf(params.ParseMode) !== -1) {
        body.parse_mode = params.ParseMode;
    }
    
    var url = 'https://api.telegram.org/bot' + params.Token + '/sendMessage';
    
    Zabbix.Log(4, '[Telegram Webhook] Telegram bildirimi gönderiliyor. URL: ' + url.replace(params.Token, '<TOKEN_REDACTED>') + ', Body: ' + JSON.stringify(body));
    
    var response = request.post(url, JSON.stringify(body));
    
    Zabbix.Log(4, '[Telegram Webhook] Yanıt durumu: ' + request.getStatus() + '. Yanıt gövdesi: ' + response);
    
    if (request.getStatus() < 200 || request.getStatus() >= 300) {
        throw 'Telegram API isteği başarısız oldu. HTTP durumu: ' + request.getStatus() + '. Yanıt: ' + response;
    }
    
    var response_json = JSON.parse(response);
    
    if (response_json.ok !== true) {
        throw 'Telegram API bir hata döndürdü: ' + response_json.description + ' (hata kodu: ' + response_json.error_code + ')';
    }
    
    // Başarıyla gönderildi
    return JSON.stringify(response_json);
    
} catch (error) {
    Zabbix.Log(3, '[Telegram Webhook] Hata: ' + error);
    
    // Zabbix, başarısız medya türü yürütmesi için bir hata fırlatılmasını bekler
    if (typeof error === 'object' && error !== null) {
        // Hata nesnesiyse, mesajını veya kendisini string'e çevir
        throw JSON.stringify({ error: String(error.message || error) });
    }
    throw String(error); // Hata zaten bir string ise doğrudan fırlat
}
        

