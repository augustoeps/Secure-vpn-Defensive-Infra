✅ **PASO 1: Instalar OpenVPN y Easy-RSA**

sudo dnf install -y openvpn easy-rsa


✅ ### **PASO 2: Crear el directorio de trabajo para Easy-RSA**

Easy-RSA necesita un entorno para trabajar. Vamos a crear un directorio llamado `openvpn-ca` en tu carpeta personal y copiar los archivos base:

mkdir -p ~/openvpn-ca
cp -r /usr/share/easy-rsa/* ~/openvpn-ca/
cd ~/openvpn-ca
