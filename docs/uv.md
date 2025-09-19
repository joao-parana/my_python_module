# Funcionalidades do uv

Ao gerenciar um projeto Python com o uv, você usará a CLI do uv muitas vezes e não se assuste pois a filosofia aqui é gerenciar todo o ciclo de vida da aplicação com esta CLI simplificando o seu workflow. Os beneficios desta abordagem são muitas como veremos adiante. Nós usamos o `uv init` para inicializar o nosso projeto e podemos, por exemplo, usar o subcomando `uv run` para executá-lo. Vamos então começar com a diversão. Suponha que queremos criar um modulo para ser compartilhado com outros desenvolvedores da companhia. Chamemos por exemplo de `my_python_module`

Execute `uv init my_python_module --lib` e depois execute `tree .`

Você verá o seguinte

```txt
.
└── my_python_module
    ├── pyproject.toml
    ├── README.md
    └── src
        └── my_python_module
            ├── __init__.py
            └── py.typed
```

Observe que foram criados vários arquivos assim que você inicializa seu projeto. Você notou que há um arquivo `__init__.py` dentro de `src/my_python_module` que também foi criado na inicialização. Este é um arquivo Python stub que contém poucas linhas de código com a definição de uma função que você já pode usar usando um comando import adequado. Então vamos fazer o build do modulo, para futura distribuição. Existe um subcomando `uv` para isso. Então, vamos em frente, limpe a tela e execute `uv build` para invocar o builder.

Você verá algo parecido com isso:

```txt
Building source distribution (uv build backend)...
Building wheel from source distribution (uv build backend)...
Successfully built dist/my_python_module-0.1.0.tar.gz
Successfully built dist/my_python_module-0.1.0-py3-none-any.whl
```

Neste momento quaisquer usuários com acesso ao arquivo `*.tar.gz` ou `*.whl` deste diretório `dist` poderão instalar e usar o modulo.

Mas como você pode usar o código desse seu projeto numa abordagem de desenvolvedor, ou seja como você invoca as funções do modulo e testa as funções existentes acrescentando novas funcionalidades no decorrer do tempo? Bem, você pode instalar no modo desenvolvedor.

use `python3 -m pip install -e .`

Depois você poderá executar `python3 -m pip list | grep my_python_module` e verá seu modulo instalado, a versão e onde foi instalado o seu código fonte.

Assim, com o modulo instalado você poderá usar no REPL do Python e adicionar os comandos `from my_python_module import hello`
`hello()` e `print(hello())` observando a saida:

```python
python3
Python 3.13.7 (main, Aug 14 2025, 11:12:11) [Clang 17.0.0 (clang-1700.0.13.3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> from my_python_module import hello
>>> hello()
'Hello from my-python-module!'
>>> print(hello())
Hello from my-python-module!
>>>
```

Vamos em frente para criar um entrypoint para nosso modulo. Chamaremos de `hello_uv` e para isso devemos:

- criar um script `src/my_python_module/main.py` para invocat a função
- adicionar uma seção `[project.scripts]` no arquivo`pyproject.toml`para que o `uv` encare como um subcomando do `uv run`

```toml
[project.scripts]
hello_uv = "my_python_module.main:hello"
```

Como isso funciona?

Se você digitar `uv run` e, em seguida, o nome do seu script (`hello_uv`) e pressionar Enter, o `uv` executa o código para você.

Como esta foi a primeira vez que o `uv run` foi usado dentro do seu projeto, você não só obtém a saída do arquivo `main.py` via função `hello`, mas também obtém duas linhas de saída que foram criadas pelo uv porque o `uv` passa a gerenciar um ambiente virtual para você.

```bash
uv run hello_uv
Using CPython 3.13.7 interpreter at: /opt/homebrew/opt/python@3.13/bin/python3.13
Creating virtual environment at: .venv
Installed 1 package in 5ms
Hello from my-python-module!
```

**OBS:** No caso de você estar com um ambiente virtual externo já ativado ele pode emitir uma Warning como esta abaixo:

```bash
uv run hello_uv
warning: `VIRTUAL_ENV=/Users/joao/.venv` does not match the project environment path `.venv` and will be ignored; use `--active` to target the active environment instead
      Built my-python-module @ file:////Users/joao/dev/python3.13/my_python_module
Installed 1 package in 3ms
Hello from my-python-module!
```

Note que você sempre pode deativar qualquer Virtual Environment já existente pois o `uv` vai gerenciar isso pra você.

Você pode ver que o `uv` mostrará a mensagem "criando um ambiente virtual em: .venv". E usando `uv run` você garante que sempre que executar seu projeto, ele estará sendo executado dentro do seu ambiente virtual. E isso permite que você tenha seu projeto isolado de outros projetos e todo o resto no seu computador. Além disso, a primeira linha de saída é apenas `uv` lembrando você da versão do Python que ele está usando, que é um feedback importante.

Então, se você listar o conteúdo da sua pasta de projeto, você pode ver que há duas coisas novas aqui. Há `.venv`, o diretório para o seu ambiente virtual. Ele foi criado pelo `uv` porque esta foi a primeira vez que você executou código no seu projeto. E há também um arquivo `uv.lock`. Este arquivo `uv.lock` é um arquivo muito interessante. Vamos limpar a tela e inspecionar seu conteúdo. No momento, é um arquivo relativamente pequeno e você quase consegue ler o arquivo e entender o que está acontecendo.
Este arquivo `uv.lock` é um arquivo de log durante toda a vida útil de um projeto, ele conterá todas as dependências do seu projeto. Por enquanto, você não tem nenhuma, apenas a linguagem Python em si. Ele conterá todas as dependências do seu projeto, até mesmo as coisas das quais você depende indiretamente (chamadas dependências transitivas). E conterá as versões exatas que você instalou no seu sistema. E a razão pela qual você quer isso é porque quando você envia esse arquivo para o seu sistema de controle de versão, outras pessoas terão acesso imediato a ele, permitindo que elas reproduzam seu ambiente virtual perfeitamente. Você ganha **reprodutibilidade** com isso. Ou seja, ao fazer com que outras pessoas reproduzam seu ambiente virtual exatamente, você garante que todos estejam usando e/ou desenvolvendo seu projeto sob as mesmas condições. Este arquivo `uv.lock` é gerenciado pelo `uv`. Você não precisa e não deve editá-lo manualmente, mas sim enviá-lo para o controle de versão o tempo todo.
Não podemos deixar de mencionar o arquivo `py.typed` que serve como uma bandeira de compatibilidade de tipo, e o `uv` o utiliza para construir um ambiente de projeto robusto, onde as ferramentas de verificação de tipo podem funcionar de forma eficaz.
O arquivo `py.typed` é crucial para a integração de ferramentas como **mypy**, **pyright** e **stubgen**, que ajudam a encontrar erros de programação antes do código ser executado.

Então, apenas uma rápida recapitulação de tudo o que nosso projeto contém. Temos arquivos de controle de versão, o diretório padrão `.git` e o arquivo `.gitignore`, que já está pré-preenchido com algumas coisas úteis para ignorar. Você tem o arquivo `.python-version` que informa ao `uv` qual é a versão padrão do Python para este projeto. Você tem o `main.py`, que é apenas um arquivo Python stub para executar sua função hello. Você pode querer colocar seu código lá ou substituí-lo por algo completamente diferente. Você tem o arquivo `pyproject.toml`, que contém metadados do projeto que você pode editar manualmente. Há o arquivo `README.md`, onde você terá a documentação básica do projeto, instruções básicas e instruções básicas de configuração. Há também o `uv.lock`, que é o arquivo de log da plataforma CREs - Cloud Runtime Environments (Ambientes de Execução em Nuvem), onde o `uv` mantém o controle de dependências e informações importantes. E, finalmente, há a pasta `.venv/`, que você pode ter visto em outras ferramentas. Você pode já ter usado essas. É uma pasta que contém seu **ambiente virtual**, onde suas dependências serão armazenadas e é gerenciado automaticamente pelo `uv`. Neste ponto já sabemos como criar um projeto, está confortável com a estrutura do projeto e entende mais ou menos o que cada arquivo faz, e precisamos ver como gerenciar dependências dentro do seu projeto.
