# Cluster Vagrant + Ansible para estudos

Para voce que, assim como eu nao pode pagar para usar a AWS entre outros provedores de Cloud, este projeto cria um cluster local para estudos do Ansible, utilizando Virtualbox e Vagrant para provisionar de maquinas virtuais.

## Fases

```
 - Provisioning => Criando as instancias para os estudos e armazenas os IP num arquivo hosts para uso do Ansible.
 - Cluster => Criação de um cluster Kubernetes e Prometheus Operator
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
A porção de rede(Classe C), o adaptador de rede em uso e o gateway padrao, são obtidos para serem utilizados pelas VM's
```bash
$REDE=`ip r | awk '/^default/ {printf "%s", $3; exit 0}' | cut -d '.' -f 1-3 | awk '{printf "%s.", $0}'`
$NIC=`ip r | awk '/^default/ {printf "%s", $5; exit 0}'`
$DEFAULT_GW=`ip r | awk '/^default/ {printf "%s", $3; exit 0}'`
```
Para o provisionamento do Ansible, é necessário informar no arquivo `/etc/hosts` o ip e o hostname das maquinas, o trecho abaixo faz essa inserção no momento da inicialização do cluster com o comando `vagrant up`
```bash
host_update_script += <<-ENTRY
    if ! egrep -q "#{ip}.*#{name}" /etc/hosts; then
      echo "#{host_entry}" | sudo tee -a /etc/hosts
    else
      echo "Entrada #{host_entry} já existe no /etc/hosts"
    fi

    if ! grep -q "#{ip}" #{inventario}; then
      echo "#{ip}" | tee -a #{inventario}
      echo "\n"
    else
      echo "Entrada #{ip} já existe no #{inventario}"
    fi
  ENTRY
```
### Ansible (2.17.1)
Provisionando cada task
```bash
ansible-playbook -i hosts main.yml
```

## License

[MIT](https://choosealicense.com/licenses/mit/)