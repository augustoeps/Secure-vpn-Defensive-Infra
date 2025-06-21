✅ **PASO 1: Instalar OpenVPN y Easy-RSA**

sudo dnf install -y openvpn easy-rsa


✅ **PASO 2: Crear el directorio de trabajo para Easy-RSA**

Easy-RSA necesita un entorno para trabajar. Vamos a crear un directorio llamado `openvpn-ca` en tu carpeta personal y copiar los archivos base:

mkdir -p ~/openvpn-ca
cp -r /usr/share/easy-rsa/* ~/openvpn-ca/
cd ~/openvpn-ca

✅ **PASO 3 (actualizado para Easy-RSA 3.x): Inicializar el entorno y definir parámetros**

cd ~/openvpn-ca/3.2.2

2. Inicializa el entorno de trabajo:

./easyrsa init-pki

✅ **PASO 4: Crear la CA (Autoridad Certificadora)**

Este paso genera la raíz de confianza (CA) que firmará los certificados del servidor y los clientes.

./easyrsa build-ca

✅ **PASO 5: Crear el certificado y clave para el servidor OpenVPN**

./easyrsa gen-req server nopass

Cuando termine, firma el certificado con la CA:

./easyrsa sign-req server server


✅ **PASO 6: Paso para generar certificado y clave para un cliente**

1)Sitúate en el directorio donde tienes EasyRSA, luego ejecuta:

./easyrsa gen-req nombre_cliente nopass

Esto generará una solicitud de certificado (CSR) y una clave privada en la carpeta pki/reqs y pki/private.

Ahora firma la solicitud con:

./easyrsa sign-req client nombre_cliente

openvpn --genkey --secret ta.key


✅ **PASO 7 :El siguiente paso es firmar la solicitud del cliente con la CA para emitir el certificado. Esto autoriza al cliente a conectarse.**

./easyrsa sign-req client cliente1

✅ **PASO 8: Ahora vamos con el siguiente paso para crear el archivo de configuración .ovpn para el cliente.**


Este archivo contendrá toda la configuración necesaria para que el cliente se conecte a tu servidor OpenVPN, incluyendo:

- Configuración de conexión (IP/puerto del servidor, protocolo, etc.)
- Certificado CA (de la autoridad)
- Certificado cliente (firmado por la CA)
- Clave privada del cliente
- TLS -auth

client
dev tun
proto udp
remote TU_IP_O_DOMINIO 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-CBC
auth SHA256
key-direction 1
verb 3

<ca>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</ca>

<cert>
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----
</cert>

<key>
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
</key>

<tls-auth>
-----BEGIN OpenVPN Static key V1-----
...
-----END OpenVPN Static key V1-----
</tls-auth>
key-direction 1



✅ **PASO 9: Abrir el puerto 1194 en el firewall del sistema y del router exterior(digi)**

sudo firewall-cmd --add-port=1194/udp --permanent
sudo firewall-cmd --reload

sudo firewall-cmd --list-ports

Ir a la pagina del router 192.168.1.1 y abrir el puerto

✅ PASO 10: Aquí te dejo un ejemplo básico y funcional del archivo `server.conf` para OpenVPN en Linux. Puedes crear o editar el archivo en `/etc/openvpn/server.conf` (o `/etc/openvpn/server/server.conf` dependiendo de tu distro):

Nota: recuerda que los certificados los cree dentro de la carpeta root al iniciar el servicio openvpn no tendra acceso a esa carpeta asi que copias esos 3 certificados y claves a una zona con acceso(en mi caso /etc/openvpn/cert

port 1194
proto udp
dev tun
tls-auth ta.key 0

ca  /etc/openvpn/cert/ca.crt
cert /etc/openvpn/cert/server.crt
key /etc/openvpn/cert/server.key
dh /etc/openvpn/dh.pem

server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt

keepalive 10 120
persist-key
persist-tun

user nobody
group nobody

push "redirect-gateway def1 bypass-dhcp"
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 8.8.4.4"

data-ciphers AES-256-CBC:AES-256-GCM:AES-128-GCM:CHACHA20-POLY1305

auth SHA256

verb 3


✅ **PASO 11: Arrancar el servicio**

sudo systemctl start openvpn-server@server.service


✅ **Paso 12: Enviar el archivo cliente.ovpn a la maquin que actuara como cliente**


1. el fichero cliente1.ovpn pasarlo al del servidor a la mv cliente

   


✅ **Paso 13: Desde la maquina cliente probar la vpn**

1. En la otra maquina descargar openvpn

3. Ejecutar 
    
    sudo openvpn --config cliente1.ovpn






