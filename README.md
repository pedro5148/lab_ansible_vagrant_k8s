# Cluster Ansible para estudos

Projeto de um cluster local para estudos do Ansible, utilizando Virtualbox e Vagrant para provisionar de maquinas virtuais.

## Fases

```
 - Provisioning => Criando as instancias para os estudos.
 - Install_k8s => Criação de um cluster Kubernetes
 - Deploy_app => Deploy de uma aplicação no cluster
```

## Dependências
### Vagrant (2.4.1)
Dentro do `Vagrantfile`, informe as configurações das VM's desejadas
```Ruby
machines = {
  "worker01" => {"memory" => "2048", "cpu" => "2", "ip" => "121", "image" => "ubuntu/jammy64"},
  "worker02" => {"memory" => "2048", "cpu" => "2", "ip" => "122", "image" => "ubuntu/jammy64"},
}
```
A porção de rede(Classe C) e o adaptador de rede em uso, são obtidos para serem utilizados pelas VM's
```bash
$REDE=`ip r | awk '/^default/ {printf "%s", $3; exit 0}' | cut -d '.' -f 1-3 | awk '{printf "%s.", $0}'`
$NIC=`ip r | awk '/^default/ {printf "%s", $5; exit 0}'`
```
Para o provisionamento do Ansible, é necessário informar no arquivo `hosts` o nome das maquinas, tendo em vista que o Vagrant usa o nome das VM's ao invés do IP, portanto é necessário informar no arquivo `/etc/hosts` o ip e o nome das VM's, o script abaixo faz essa inserção no momento da inicialização do cluster com o comando `vagrant up`
```bash
machines.each do |name, conf|
  ip = "#{$REDE}#{conf["ip"]}"
  host_entry = "#{ip}\t#{name}"
  host_update_script += <<-ENTRY
    if ! grep -q "#{ip}" /etc/hosts; then
      echo "#{host_entry}" | sudo tee -a /etc/hosts
    else
      echo "Entrada #{host_entry} já existe no /etc/hosts"
    fi
  ENTRY
end

host_update_script += <<-SCRIPT
  } > /dev/null
SCRIPT

File.write('/tmp/update_hosts.sh', host_update_script)
system('bash /tmp/update_hosts.sh')
```
### Ansible (2.16.8)
Para provisionar o cluster, no Vagrantfile é adicionado as linhas abaixo para dizer ao Vagrant onde esta o playbook e o arquivo hosts provisionar nas VM's
```bash
config.vm.provision "ansible" do |ansible|
    ansible.playbook = "$YOUR_PATH/cluster-k8s/provisioning/main.yml"
    ansible.compatibility_mode = "2.0"
    ansible.inventory_path = "$YOUR_PATH/cluster-k8s/provisioning/hosts"
end
```

## License

[MIT](https://choosealicense.com/licenses/mit/)