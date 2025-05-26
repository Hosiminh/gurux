1. Örnek Klasörü Klonlayıp Kullanmak
GitHub deposunu alın:

bash
Kopyala
Düzenle
cd ~/dlms
git clone https://github.com/Gurux/Gurux.DLMS.Python.git
Örnek klasöre gidin ve PYTHONPATH’e ekleyin:

bash
Kopyala
Düzenle
cd Gurux.DLMS.Python/Gurux.DLMS.Client.Example.python
export PYTHONPATH="$PWD:$PYTHONPATH"
Sanal ortamınızı aktif edip bağımlılıkları yükleyin:

bash
Kopyala
Düzenle
source ~/dlms/venv/bin/activate
pip install -r requirements.txt
Artık aşağıdaki gibi import edebilirsiniz:

python
Kopyala
Düzenle
from GXDLMSReader import GXDLMSReader
Örnek main.py’yi doğrudan komut satırından da çalıştırabilirsiniz:

bash
Kopyala
Düzenle
python main.py \
  -S /dev/serial0:9600:8None1 \
  -g "1.8.0.255:2" \
  -o device.xml
Bu, OBIS 1.8.0 (toplam aktif enerji) değerini okuyacaktır 
Gurux
.

2. GXDLMSClient ile Doğrudan Okuma (Reader Olmadan)
Eğer GXDLMSReader’ı projenize taşımak istemezseniz, yalnızca GXDLMSClient ve GXSerial kullanarak şöyle ilerleyebilirsiniz:

python
Kopyala
Düzenle
from gurux_serial import GXSerial
from gurux_dlms import GXDLMSClient
from gurux_common import Authentication

# 1) Seri portu açın
media = GXSerial(port="/dev/serial0")
media.baudRate = 9600
media.parity   = 0      # 0=None
media.dataBits = 8
media.stopBits = 1
media.open()

# 2) DLMS istemcisini başlatın (Logical Name referencing)
client = GXDLMSClient(useLogicalNameReferencing=True)
client.clientAddress = 1
client.serverAddress = 1
client.authentication = Authentication.NONE

# 3) HDLC bağlantıyı kurun
#    (SNRM/AARQ el sıkışma paketlerini gönder/oku)
client.snrmRequest(media)
client.aarqRequest(media)

# 4) OBIS 1.8.0’daki (aktif enerji) attribute=2’yi okuyun
data = client.getAttribute("1.8.0", 2)
media.send(data)
reply = media.receive()
value = client.parseReadData(reply)
print("Toplam Aktif Enerji (kWh):", value)

# 5) Kapatın
media.close()
Bu yol ile ekstra bir Reader sınıfı import etmeye gerek kalmadan doğrudan metre okumalarınızı yapabilirsiniz.
