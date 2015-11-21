#!/bin/sh
$script = <<SCRIPT
echo "Update packages"
sudo killall -9 java > /dev/null 2>&1
sudo apt-get update > /dev/null 2>&1
echo "Install Java and Elasticsearch 2"
echo oracle-java8-installer shared/accepted-oracle-license-v1-1 select true | sudo /usr/bin/debconf-set-selections
sudo apt-get -y install curl software-properties-common > /dev/null 2>&1
curl -s http://packages.elasticsearch.org/GPG-KEY-elasticsearch | apt-key add -
echo deb http://packages.elasticsearch.org/elasticsearch/2.x/debian stable main > /etc/apt/sources.list.d/elasticsearch-2.x.list
sudo add-apt-repository ppa:webupd8team/java > /dev/null 2>&1
sudo apt-get update > /dev/null 2>&1
sudo apt-get -y install nc elasticsearch oracle-java8-installer > /dev/null 2>&1

echo "Install Logstash"
LOGSTASH_HOME=/opt/logstash
LOGSTASH_PACKAGE=logstash-2.0.0.tar.gz

mkdir -p ${LOGSTASH_HOME} \
 && curl -s -O https://download.elasticsearch.org/logstash/logstash/${LOGSTASH_PACKAGE} \
 && tar xzf ${LOGSTASH_PACKAGE} -C ${LOGSTASH_HOME} --strip-components=1 \
 && rm -f ${LOGSTASH_PACKAGE} \
 && groupadd -r logstash \
 && useradd -r -s /usr/sbin/nologin -d ${LOGSTASH_HOME} -c "Logstash service user" -g logstash logstash \
 && mkdir -p /var/log/logstash /etc/logstash/conf.d \
 && chown -R logstash:logstash ${LOGSTASH_HOME} /var/log/logstash > /dev/null 2>&1

cp /vagrant/logstash-init /etc/init.d/logstash
sed -i -e 's#^LS_HOME=$#LS_HOME='$LOGSTASH_HOME'#' /etc/init.d/logstash \
 && chmod +x /etc/init.d/logstash

echo "Install Kibana"
KIBANA_HOME=/opt/kibana
KIBANA_PACKAGE=kibana-4.2.0-linux-x64.tar.gz

mkdir -p ${KIBANA_HOME} \
 && curl -s -O https://download.elasticsearch.org/kibana/kibana/${KIBANA_PACKAGE} \
 && tar xzf ${KIBANA_PACKAGE} -C ${KIBANA_HOME} --strip-components=1 \
 && rm -f ${KIBANA_PACKAGE} \
 && groupadd -r kibana \
 && useradd -r -s /usr/sbin/nologin -d ${KIBANA_HOME} -c "Kibana service user" -g kibana kibana \
 && chown -R kibana:kibana ${KIBANA_HOME} > /dev/null 2>&1

/opt/kibana/bin/kibana plugin --install elasticsearch/marvel/latest

cp /vagrant/kibana-init /etc/init.d/kibana
sed -i -e 's#^KIBANA_HOME=$#KIBANA_HOME='$KIBANA_HOME'#' /etc/init.d/kibana \
 && chmod +x /etc/init.d/kibana

#su -c "/vagrant/setup.sh" vagrant

ES_BIN=/usr/share/elasticsearch

echo "Install Elasticsearch plugins"
su -c "$ES_BIN/bin/plugin install license > /dev/null 2>&1" elasticsearch
su -c "$ES_BIN/bin/plugin install shield > /dev/null 2>&1" elasticsearch

rm -f /var/run/elasticsearch/elasticsearch.pid /var/run/logstash.pid \
  /var/run/kibana4.pid

#sudo /bin/systemctl daemon-reload
#  sudo /bin/systemctl enable elasticsearch.service
#  sudo /bin/systemctl start elasticsearch.service

service elasticsearch start
service logstash start



# wait for elasticsearch to start up
# - https://github.com/elasticsearch/kibana/issues/3077
while ! nc -q 1 localhost 9200 </dev/null; do echo "wait"; sleep 2; done
#counter=0
#while [ "$(nc -z localhost 9200)" -a $counter -lt 30  ]; do
#  sleep 1
#  ((counter++))
#  echo "waiting for Elasticsearch to be up ($counter/30)"
#done

#cat /var/log/elasticsearch/elasticsearch.log

service kibana start


SCRIPT
# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  # set to false, if you do NOT want to check the correct VirtualBox Guest Additions version when booting this box
  if defined?(VagrantVbguest::Middleware)
    config.vbguest.auto_update = true
  end

  config.vm.box = "ubuntu/trusty64" #"puppetlabs/ubuntu-14.04-64-puppet"
  config.vm.network :forwarded_port, guest: 5601, host: 5601
  config.vm.network :forwarded_port, guest: 9200, host: 9200
  config.vm.network :forwarded_port, guest: 9300, host: 9300

  config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--cpus", "2", "--memory", "2048"]
  end

  config.vm.provider "vmware_fusion" do |v, override|
     ## the puppetlabs ubuntu 14-04 image might work on vmware, not tested?
    v.provision "shell", path: 'ubuntu.sh'
    v.box = "phusion/ubuntu-14.04-amd64"
    v.vmx["numvcpus"] = "2"
    v.vmx["memsize"] = "2048"
  end
  config.vm.provision "shell", inline: $script
  #config.vm.provision "shell", path: 'setup.sh'
  #config.vm.provision "puppet",  manifest_file: "default.pp"
end
