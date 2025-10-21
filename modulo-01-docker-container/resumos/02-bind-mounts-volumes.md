## Persistência de Dados e a Natureza Efêmera dos Containers

### 1. Entendendo a Perda de Dados Após a Remoção do Container

Por padrão, tudo o que acontece dentro de um container Docker é efêmero. Ao remover um container, todos os dados e alterações feitas dentro dele são perdidos. Isso é útil para manter a consistência e portabilidade, mas pode ser problemático quando precisamos persistir dados, como em bancos de dados ou aplicações que armazenam informações de usuários.

#### Camadas de Leitura e Escrita nos Containers

Os containers Docker utilizam um sistema de arquivos em camadas, conhecido como OverlayFS. Quando um container é iniciado a partir de uma imagem, o Docker cria uma camada de leitura e escrita sobre as camadas somente leitura da imagem. Todas as alterações feitas no container são gravadas nessa camada superior.

#### Introdução ao OverlayFS e Funcionamento das Camadas

* **OverlayFS**: É um sistema de arquivos unificador que permite sobrepor múltiplos sistemas de arquivos.
* **Camadas de Imagem:** São camadas somente leitura que compõem a imagem Docker.
* **Camada de Container:** É a camada de leitura e escrita criada quando o container é iniciado.

Esse sistema permite que múltiplos containers compartilhem a mesma imagem base sem interferir uns nos outros, economizando espaço e recursos.

### 2. Introdução à Persistência de Dados

#### Por que Precisamos Persistir Dados?

Em muitos casos, precisamos que os dados sobrevivam além do ciclo de vida de um container. Por exemplo:

* Bancos de dados que armazenam informações críticas.
* Aplicações que geram logs importantes.
* Sites que permitem uploads de arquivos.

#### Conceitos de Volumes e Bind Mounts no Docker

* **Volumes:**
    * Gerenciados pelo Docker.
    * Armazenados em um local específico no sistema de arquivos do Docker.
    * Podem ser locais ou remotos (usando drivers de volume).
    * São a maneira recomendada para persistir dados em produção.
* **Bind Mounts:**
    * Montam um diretório ou arquivo do sistema de arquivos do host no container.
    * Oferecem mais flexibilidade durante o desenvolvimento.
    * Dependem da estrutura do host, o que pode afetar a portabilidade.

### 3. Utilizando Bind Mounts para Compartilhar Diretórios

#### Configurando Bind Mounts com a Flag v ou -mount

* Usando a Flag `-v`

A sintaxe básica para um bind mount com `-v` é:

```
docker run -v [caminho_do_host]:[caminho_do_container] [imagem]
```

* Usando a Flag `-mount`

A sintaxe com `--mount` é mais explícita:

```
docker run --mount type=bind,source=[caminho_do_host],target=[caminho_do_container] [imagem]
```

#### Exemplo Prático: Executando Nginx com Bind Mount

* **Passo 1:** Criar um Diretório no Host

Crie um diretório para os arquivos do Nginx:

```
mkdir ~/my_nginx_html
```

* **Passo 2:** Criar um Arquivo index.html

Crie um arquivo HTML simples:

```
echo "<h1>Hello Docker!</h1>" > ~/my_nginx_html/index.html
```

* **Passo 3:** Executar o Nginx com Bind Mount

Usando `-v` e `$(pwd)` para obter o caminho atual:

```
docker run -d -p 8080:80 -v $(pwd)/my_nginx_html:/usr/share/nginx/html nginx
```

> Nota: Certifique-se de estar no diretório onde o my_nginx_html está localizado ao usar `$(pwd)`.

* **Passo 4:** Verificar no Navegador

Acesse http://localhost:8080 e você deverá ver "Hello Docker!".

#### Editando Arquivos no Host e Vendo as Alterações no Container

* **Passo 1:** Alterar o index.html no Host

Edite o arquivo:

```
echo "<h1>Content Updated!</h1>" > ~/my_nginx_html/index.html
```

* **Passo 2:** Atualizar o Navegador

Recarregue a página em http://localhost:8080 e veja a alteração refletida.

#### Demonstrando a Efemeridade sem Bind Mounts

* **Passo 1:** Remover o Container

```
docker rm -f [container_id ou nome]
```

* **Passo 2:** Recriar o Container sem Bind Mount

```
docker run -d -p 8080:80 nginx
```

* **Passo 3:** Verificar se as Alterações Foram Perdidas

Acesse http://localhost:8080 e note que as alterações feitas anteriormente não estão mais presentes.

#### Demonstrando a Persistência com Bind Mounts

* **Passo 1:** Remover o Container

```
docker rm -f [container_id ou nome]
```

* **Passo 2:** Recriar o Container com Bind Mount

```
docker run -d -p 8080:80 -v $(pwd)/my_nginx_html:/usr/share/nginx/html nginx
```

* **Passo 3:** Verificar a Persistência das Alterações

Acesse http://localhost:8080 e confirme que o conteúdo alterado permanece.

#### Dicas sobre Bind Mounts

* **Caminho Absoluto:** Sempre use caminhos absolutos ou `$(pwd)` para evitar problemas.
* **Permissões:** Certifique-se de que o Docker tem permissão para acessar o diretório.
* **Compatibilidade:** Os bind mounts dependem do sistema de arquivos do host, o que pode afetar a portabilidade entre diferentes sistemas.

### 4. Gerenciando Volumes Docker

#### Tipos de Volumes

* **Volumes Locais:** Armazenados no sistema de arquivos do host, gerenciados pelo Docker.
* **Volumes Remotos:**  Utilizam drivers de volume para armazenar dados em soluções de rede, como NFS, Azure File Storage, AWS EFS, etc.

#### Criando e Gerenciando Volumes com docker volume

* Criar um Volume

```
docker volume create my_volume
```

* Listar Volumes

```
docker volume ls
```

![alt text](/modulo-01-docker-container/images/image-2.png)
> Volume docker armazendo localmente

* Inspecionar um Volume

```
docker volume inspect my_volume
```

Principais Linhas do inspect:

* **"Mountpoint":**  Local no sistema de arquivos onde o volume está armazenado.
* **"Driver":**  O driver usado (geralmente "local").
* **"Labels":** Metadados que podem ser atribuídos ao volume.

Exemplo de saída:

```
[
    {
        "CreatedAt": "2023-10-10T12:34:56Z",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my_volume/_data",
        "Name": "my_volume",
        "Options": {},
        "Scope": "local"
    }
]
```

* Remover um Volume

```
docker volume rm my_volume
```

#### Montando Volumes em Containers

* **Passo 1:** Executar Nginx com um Volume

```
docker run -d -p 8081:80 -v my_volume:/usr/share/nginx/html nginx
```

![alt text](/modulo-01-docker-container/images/image-3.png)
> Aqui ele cria um container usando um volume como bind mount, onde ele encontra a pasta (volume) e joga o conteudo do container nele.

* **Passo 2:** Copiar Arquivos para o Volume

Copie o arquivo index.html para o container:

```
docker cp ~/my_nginx_html/index.html [container_id ou nome]:/usr/share/nginx/html
```


![alt text](/modulo-01-docker-container/images/image-4.png)
> 

![alt text](/modulo-01-docker-container/images/image-5.png)
> Nota: Como estamos usando um volume, o arquivo será persistido mesmo após a remoção do container.


* **Passo 3:** Verificar no Navegador

Acesse http://localhost:8081 para ver o conteúdo.

* **Passo 4:** Remover e Recriar o Container

```
docker rm -f [container_id ou nome]
docker run -d -p 8081:80 -v my_volume:/usr/share/nginx/html nginx
```

* **Passo 5:** Confirmar Persistência dos Dados

Acesse novamente http://localhost:8081 e veja que o conteúdo permanece.

![alt text](/modulo-01-docker-container/images/image-6.png)
> Mostrando que é possivel criar varios container que utiliza o mesmo arquivo, no entanto isso não é recomendavel pois pode gerar concorrência.

#### Comparação entre Volumes e Bind Mounts

* **Volumes**
    * **Gerenciados pelo Docker:** O Docker controla onde os dados são armazenados.
    * **Independência do Host:** Não dependem da estrutura de diretórios do host.
    * **Segurança:** Oferecem melhor isolamento, já que o host não acessa diretamente os dados.
    * **Backup e Restauração:** Facilmente gerenciáveis para backup e migração.
    * **Compatibilidade com Drivers de Volume:** Podem utilizar drivers para armazenar dados em locais remotos ou em rede.
* **Bind Mounts**
    * **Dependência do Host:** Dependem da estrutura de diretórios e sistema de arquivos do host.
    * **Flexibilidade:** Permitem acesso direto aos arquivos do host, útil em ambientes de desenvolvimento.
    * **Risco de Segurança:** Maior risco se não configurados corretamente, pois dão ao container acesso direto ao sistema de arquivos do host.
    * **Desempenho:** Podem ter desempenho variável dependendo do sistema de arquivos e configuração.

#### Quando Usar Cada Um

* **Volumes**
    * **Produção:** Maior segurança e independência do host.
    * **Persistência de Dados Sensíveis:** Bancos de dados, dados de aplicação.
    * Quando se deseja usar drivers de volume para armazenamento em rede ou em nuvem.
* **Bind Mounts**
    * **Desenvolvimento:** Facilita a edição de arquivos e atualização em tempo real.
    * **Casos Especiais:** Quando é necessário acessar arquivos específicos do host.

### 5. Backup e Restauração de Volumes

Para garantir a persistência dos dados armazenados em volumes, é uma prática recomendada criar backups regulares, especialmente em ambientes de produção.


#### Backup de um Volume

```
docker run --rm -v my_volume:/data -v $(pwd):/backup busybox tar czf /backup/backup.tar.gz /data
```

* `docker run`: Executa um novo container.
* `-rm`: Remove o container automaticamente ao final do processo.
* `v my_volume:/data`: Monta o volume my_volume no caminho /data dentro do container.
* `v $(pwd):/backup`: Monta o diretório atual do host (`$(pwd)`) no caminho /backup dentro do container.
* `busybox`: Uma imagem de contêiner leve usada para executar comandos Unix básicos.
* `tar czf /backup/backup.tar.gz /data`:
    * Cria um arquivo compactado backup.tar.gz com o conteúdo do volume my_volume e o armazena no diretório atual do host.

#### Restauração de um Volume

```
docker run --rm -v my_volume:/data -v $(pwd):/backup busybox tar xzf /backup/backup.tar.gz -C /
```

* `docker run`: Executa um novo container.
* `-rm`: Remove o container automaticamente ao final do processo.
* `v my_volume:/data`: Monta o volume my_volume no caminho /data dentro do container.
* `v $(pwd):/backup`: Monta o diretório atual do host, onde o arquivo de backup está localizado.
* `tar xzf /backup/backup.tar.gz -C /`:
    * Extrai o conteúdo de /backup/backup.tar.gz no diretório raiz do container.
    * O arquivo backup.tar.gz inclui o caminho /data, que restaura os dados diretamente no volume my_volume