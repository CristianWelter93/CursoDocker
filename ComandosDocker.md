# COMANDOS MAIS BÁSICOS
* docker version - exibe a versão do docker.
* docker run NOME_DA_IMAGEM - cria um container com a respectiva imagem passada como parâmetro.
* docker inspect {id} : exibe informções do container
* docker login : dar login no dockerhub para subir o seu container
* docker push {nome container}: subir o container. Deve obdecer ao padrão criador/nomecontainer, o criador deve ser o mesmo nome de usuario do dockerhub
* docker pull {nome container}
* docker ps : exibe constainers que estão ativos no momento;
* docker ps -a : exibe todos os constainers mesmo os não ativos; 

# Inicialização e Desativação
* docker run -it {ubuntu ou outro nome}: atrelo o terminal ao container. o terminal "passsa a ser" o de dentro do container do ubuntu;
* docker run -d {nome_da_imagem}: roda sem travar o terminal;
* docker run -d --name {apelido} {nome_da_imagem}: coloca um apelido pra não precisar do id;
* docker run -d -P --name {NOME} {nome_da_imagem} - ao executar, dá um nome ao container.
* docker start {id do container}: inicializo um container;
* docker start -a: inicio e atrelo o terminal ao do container;
* docker stop {id container}: para o container;
* docker stop -t 0 {id container}: não espera os 10 segundos para parar o container;
* docker stop $(docker ps -q): para todos os containers ;

# Remoção de constainers 
* docker rm {id}: remover o container;
* docker container prune: limpa todos os containers inativos;
* docker rmi {id}: remover imagem;

# Portas
* docker-machine ip: descobrir ip da maquina virtual para quando usar o toolbox
* docker run -d -P {nome_da_imagem}: linkar uma porta interna do container para o meu computador;
* docker run -d -p {portadesejada} {nome_da_imagem}: determinar uma porta interna do container para o meu computador;
* docker port {id}: visualizar mapeamento

# Salvar os dados dos constainers separadamente (Volumes)
* docker run -v "{nome}" {container} :criando container com volume;
* docker run -v "{caminho do computador: caminho do container}" {container} : configurar em qual caminho queremos salvar o volume;
* docker run -d -p {portaReal:portacontainer} -v "{caminho pc: container}" -w "{onde executar}" {container e comandos}: qual diretorio que é pra executar os comandos;
* Exemplo estando na pasta do arquivo pelo cmd: docker run -d -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start;

# Criar propria imagem
## Criar arquivo Dockerfile e colocar nele:
* Na linha do Copy, se tiver na pasta coloca . para copiar tudo  de onde ta e qual seria o destino EX: COPY . /var/www
* Na linha RUN executa algum comando como instalar as dependências, EX: RUN npm install :instalar dependências
* ENTRYPOINT :comando a ser executado assim que carregado o container, EX: npm start
* EXPOSE {numero da porta} :dize qual porta o container utiliza
* Exemplo da estrutura básica do arquivo:

FROM {imagem de referencia}:latest
MAINTAINER {nome da pessoa que a esta mantendo}
COPY {caminho origem} {caminho de destino}
WORKDIR {pasta onde se baseia}
RUN {comando}
ENTRYPOINT {comando}
EXPOSE {numero da porta}

## Executa então o comando docker build -f Caminho/Dockerfile -t {usuarioquecriou/nomedaimagem} .

# Networkig no Docker, comunicação entre os containers
* docker network create --driver bridge minha-rede : criar a propria rede, usando o driver bridge
* docker network ls : exibe as redes disponíveis
* docker run -it --name meu-container-de-ubuntu --network minha-rede ubuntu: apelida um container ubuntu e o coloca na rede criada

# Subir múltiplos containers: Docker Compose

* services são quains constainers serão construidos
* build indica quando um arquivo tem configurações especificas: 
	-indico onde estar o arquivo de configuração -
	indico em qual pasta (contexto)
	
* caso não exista configurações especificas utilizo apenas o imagem, dando um apelido a ela
* ports quais portas serão utilizadas 
* networks indica em qual rede o container será colocado
* depends_on indica qual a ordem de construção, quem precisa esperar para ser construído.
* network indica que será realizada a criação de uma rede

## Exemplo de Arquivo de configuração

version: '3'
services:
  nginx:
    build:
      dockerfile: ./docker/nginx.dockerfile
      context: .
    image: cristianwelter/nginx
    container_name: nginx
    ports:
      - "80:80"
    networks: 
      - production-network
    depends_on:
      - "node1"
      - "node2"
      - "node3"
  mongodb:
    image: mongo
    networks: 
      - production-network

  node1:
    build:
      dockerfile: ./docker/alura-books.dockerfile
      context: .
    image: cristianwelter/alura-books
    container_name: alura-books-1
    ports:
      - "3000"
    networks: 
      - production-network
    depends_on:
      - "mongodb"

  node2:
    build:
      dockerfile: ./docker/alura-books.dockerfile
      context: .
    image: cristianwelter/alura-books
    container_name: alura-books-2
    ports:
      - "3000"
    networks: 
      - production-network
    depends_on:
      - "mongodb"

  node3:
    build:
      dockerfile: ./docker/alura-books.dockerfile
      context: .
    image: cristianwelter/alura-books
    container_name: alura-books-3
    ports:
      - "3000"
    networks: 
      - production-network
    depends_on:
      - "mongodb"
networks: 
  production-network:
    driver: bridge

* docker-compse up :executar o yml com todas as configurações inseridas