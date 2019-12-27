#!/bin/bash
echo "America/Sao_Paulo" > /etc/timezone
ln -fs /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime > /dev/null 2>&1
dpkg-reconfigure --frontend noninteractive tzdata > /dev/null 2>&1
ip=$(ip addr | grep 'inet' | grep -v inet6 | grep -vE '127\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | grep -o -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | head -1)
if [[ "$ip" = "" ]]; then
	ip=$(wget -qO- ipv4.icanhazip.com)
fi
clear
rm /root/inst > /dev/null 2>&1
rm /root/install > /dev/null 2>&1
echo ""
sleep 3
echo ""
echo -e "\033[1;31mATENÃ‡ÃƒO \033[1;32mINFORME SEMPRE A MESMA SENHA"
echo -e "\033[1;32mSEMPRE COMFIRME AS QUESTÃ”ES COM\033[1;37m Y"
echo ""
echo -e "\033[1;36mINICIANDO INSTALAÃ‡ÃƒO"
echo ""
echo -e "\033[1;33mBAIXANDO ATUALIZAÃ‡Ã•ES"
apt-get update > /dev/null 2>&1
apt-get upgrade -y -q > /dev/null 2>&1
echo ""
echo -e "\033[1;36mINSTALANDO O APACHE2\033[0m"
apt-get install apache2 -y > /dev/null 2>&1
apt-get install cron curl unzip -y > /dev/null 2>&1
echo ""
echo -e "\033[1;36mINSTALANDO DEPENDÃŠNCIAS\033[0m"
apt-get install php5 libapache2-mod-php5 php5-mcrypt -y > /dev/null 2>&1
service apache2 restart 
echo ""
echo -e "\033[1;36mINSTALANDO O MySQL\033[0m"
echo ""
sleep 1
apt-get install mysql-server -y 
echo ""
clear
echo -e "\033[1;31mATENÃ‡ÃƒO \033[1;32mINFORME SEMPRE A MESMA SENHA QUANDO PEDIR"
echo -e "\033[1;32mSEMPRE COMFIRME AS QUESTÃ•ES COM\033[1;37m Y"
echo ""
echo -ne "\033[1;33mEnter, Para Prosseguir!\033[1;37m"; read
mysql_install_db
mysql_secure_installation
clear
echo -e "\033[1;36mINSTALANDO O PHPMYADMIN\033[0m"
echo ""
echo -e "\033[1;31mATENÃ‡ÃƒO \033[1;33m!!!"
echo ""
echo -e "\033[1;32mSELECIONE A OPÃ‡ÃƒO \033[1;31mAPACHE2 \033[1;32mCOM A TECLA '\033[1;33mESPAÃ‡O\033[1;32m'"
echo ""
echo -e "\033[1;32mSELECIONE \033[1;31mYES\033[1;32m NA OPÃ‡ÃƒO A SEGUIR (\033[1;36mdbconfig-common\033[1;32m)"
echo -e "PARA CONFIGURAR O BANCO DE DADOS"
echo ""
echo -e "\033[1;32mLEMBRE-SE INFORME A MESMA SENHA QUANDO SOLICITADO"
echo ""
echo -ne "\033[1;33mEnter, Para Prosseguir!\033[1;37m"; read
apt-get install phpmyadmin -y
php5enmod mcrypt
service apache2 restart
ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin
apt-get install libssh2-1-dev libssh2-php -y > /dev/null 2>&1
if [ "$(php -m |grep ssh2)" = "ssh2" ]; then
  true
else
  clear
  echo -e "\033[1;31mERRO CRÃTICO\033[0m"
  exit
fi
apt-get install php5-curl > /dev/null 2>&1
service apache2 restart
clear
echo ""
echo -e "\033[1;31mATENÃ‡ÃƒO \033[1;33m!!!"
echo ""
echo -ne "\033[1;32mINFORME A MESMA SENHA\033[1;37m: "; read senha
echo -e "\033[1;32mOK\033[1;37m"
sleep 1
mysql -h localhost -u root -p$senha -e "CREATE DATABASE plus"
clear
echo -e "\033[1;36mFINALIZANDO INSTALAÃ‡ÃƒO\033[0m"
echo ""
echo -e "\033[1;31mAguarde, pode demorar \033[1;33m..."
echo ""
cd /var/www/html/
echo " "
wget https://www.dropbox.com/s/trao9tpwziaccam/html.zip?dl=0 > /dev/null 2>&1
sleep 1
unzip html.zip > /dev/null 2>&1
rm -rf html.zip index.html
service apache2 restart
sleep 1
if [[ -e "/var/www/html/pages/system/pass.php" ]]; then
sed -i "s;suasenha;$senha;g" /var/www/html/pages/system/pass.php > /dev/null 2>&1
fi
sleep 1
cd
wget http://sshbrasil.com.br/Scripts/web/plus.sql > /dev/null 2>&1 
sleep 1
if [[ -e "$HOME/plus.sql" ]]; then
    mysql -h localhost -u root -p$senha --default_character_set utf8 plus < plus.sql
    rm /root/plus.sql
else
    clear
    echo -e "\033[1;31mERRO AO IMPORTAR BANCO DE DADOS\033[0m"
    sleep 2
	rm /root/install > /dev/null 2>&1
    exit
fi
echo '* * * * * root /usr/bin/php /var/www/html/pages/system/cron.php' >> /etc/crontab
echo '* * * * * root /usr/bin/php /var/www/html/pages/system/cron.ssh.php ' >> /etc/crontab
echo '* * * * * root /usr/bin/php /var/www/html/pages/system/cron.sms.php' >> /etc/crontab
echo '* * * * * root /usr/bin/php /var/www/html/pages/system/cron.online.ssh.php' >> /etc/crontab
echo '10 * * * * root /usr/bin/php /var/www/html/pages/system/cron.servidor.php' >> /etc/crontab
/etc/init.d/cron reload > /dev/null 2>&1
/etc/init.d/cron restart > /dev/null 2>&1
chmod 777 /var/www/html/admin/pages/servidor/ovpn
chmod 777 /var/www/html/admin/pages/download
chmod 777 /var/www/html/admin/pages/faturas/comprovantes
service apache2 restart
sleep 1
clear
echo -e "\033[1;32mPAINEL INSTALADO COM SUCESSO!"
echo ""
echo -e "\033[1;36mSEU PAINEL ADM\033[1;37m $ip/admin\033[0m"
echo -e "\033[1;36mUSUÃRIO\033[1;37m admin\033[0m"
echo -e "\033[1;36mSENHA\033[1;37m admin\033[0m"
echo ""
echo -e "\033[1;33mAltere a senha quando logar no painel\033[0m"
cat /dev/null > ~/.bash_history && history -c
rm /root/install > /dev/null 2>&1
rm /root/inst > /dev/null 2>&1
sleep 1000
