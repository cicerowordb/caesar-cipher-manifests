# Manifestos do Cifra de César

Este repositório contém manifestos Kubernetes para a implantação da aplicação Cifra de César, projetada para demonstrar técnicas básicas de criptografia usando clusters Kubernetes.

Para o código-fonte e mais detalhes sobre a própria aplicação, consulte nosso [repositório de código](https://github.com/cicerowordb/caesar-cipher-code).

Todo o cluster é implantado em uma única instância da AWS executando uma distribuição Kubernetes K3S leve, ideal para fins educacionais e experimentos em pequena escala como este.

### Acesso e permissões

* Exponha as portas TCP/5001 e TCP/6443 de suas instâncias usando grupos de segurança da AWS.

### Exportar acesso à configuração do cluster

Para exportar a configuração do seu cluster Kubernetes para acesso externo, use a seguinte sequência de comandos para substituir o IP local pelo endereço IPv4 público da sua instância. Isso permite que você gerencie seu cluster remotamente via kubectl ou outras ferramentas de gerenciamento Kubernetes.

```bash
INSTANCE_PUBLIC_IPV4=$(curl http://169.254.169.254/latest/meta-data/public-ipv4)
k3s kubectl config view --raw --output json \
    |tr -d " \n" \
    |sed "s/127.0.0.1/$INSTANCE_PUBLIC_IPV4/"
```

O primeiro comando obtém o IPv4 público de sua instância da AWS e define uma nova variável de ambiente. O segundo comando obtém a configuração do K3S e substitui o IP de loopback pelo IP público da instância. O resultado é um JSON para ser incluído dentro de um segredo do GitHub Actions: `secrets.KUBE_CONFIG`. Configure este segredo dentro do seu repositório antes de executar o pipeline.

### Criar novo segredo para o registro privado do Docker Hub

Para permitir que o Kubernetes puxe imagens de um registro privado do Docker Hub, crie um segredo Kubernetes com suas credenciais do Docker conforme mostrado abaixo. Este segredo será usado pelos pods Kubernetes para autenticar no Docker Hub e habilitar a puxada de imagens privadas necessárias para sua implantação.

```bash
kubectl create secret docker-registry my-docker-credentials \
    --docker-server=https://index.docker.io/v1/ \
    --docker-username=<YOUR_DOCKER_USERNAME> \
    --docker-password=<YOUR_DOCKER_PASSWORD> \
    --docker-email=<YOUR_DOCKER_EMAIL>
```

# Pipeline de Implantação

Este repositório inclui um pipeline de implantação contínua (CD) configurado para ser acionado por pushes e pull requests na branch main. Verifique o arquivo YAML `.github/workflows/cd.yaml` para entender todos os passos.

O pipeline compreende várias etapas, começando com a instalação do `kubectl`, que baixa uma versão específica. As etapas subsequentes incluem a exportação da configuração do Kubernetes de um segredo do GitHub para um arquivo `~/.kube/config` para autenticação em um cluster Kubernetes.

O código é então verificado a partir da branch que acionou a ação. O pipeline realiza simulações locais e remotas das configurações do Kubernetes para validá-las antes de finalmente aplicar as configurações ao cluster. Isso garante que cada alteração enviada à branch main seja automaticamente testada quanto à sintaxe e viabilidade no ambiente do cluster antes da implantação, aumentando a confiabilidade e a automação no processo de implantação.

### Problemas conhecidos

A opção `--insecure-skip-tls-verify` é usada com o kubectl para instruir a ferramenta a ignorar a verificação dos certificados TLS. Isso é necessário porque a instalação do K3S usa os IPs privados e de loopback no certificado (por padrão) e estamos usando o IP público para acessar a API. Para evitar etapas adicionais explicando como substituir o certificado da API do K3S, estamos simplesmente ignorando problemas de certificado.
