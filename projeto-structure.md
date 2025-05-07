A primeira coisa que vamos fazer é preencher os dados pessoais para nos matricularmos. Aqui já te adianto, essa primeira parte da tela vai enviar os dados para um microsserviço. Então, vamos lá!

[00:33] O nome completo vai ser "Vinicius Dias" e esse vai ser o e-mail. Beleza, vou para o próximo. Já enviamos os dados para um microsserviço, só que agora precisamos mandar para um microsserviço de pagamentos. Então vamos preencher todos os dados que o microsserviços de pagamento precisa.

[00:48] Vou colocar um CPF qualquer. O titular do cartão vai ser eu mesmo, então deixe-me digitar certo. Beleza, é um cartão fictício. Acho que digitei número de mais. Já temos validações. Deu para você ver. Tem o código de segurança. Beleza!

[01:04] Quando fizer isso, o que vai acontecer? Esses dados vão ser enviados para o microsserviço de pagamento. Mas repare que ele já nos exibe que o pedido foi efetuado e que o pagamento está sendo processado.

[01:16] Então uma tarefa de plano de fundo está rodando. Isso vai levar cerca de 3 segundos. É uma tarefa fictícia, então já programei um tempo. Depois desse tempo vai chegar um e-mail na caixa de entrada. Já, já eu vou te mostrar o e-mail.

[01:29] Agora que provavelmente já passaram esses 30 segundos e sei que posso ir na minha página de login usar aquele e-mail e a senha que eu sei que o microsserviço criou. A partir disso, se tudo der certo, já posso fazer o login.

[01:43] E está lá! Tenho acesso ao microsserviço acadêmico. Ele permite que eu entre ou não e me mostra os cursos disponíveis. A partir desses cursos eu posso marcar algum desses como assistido. Então nós já temos uma aplicação desenvolvida com três microsserviços e vamos entender quais são os componentes que fazem parte de cada um desses microsserviços.

Sobre o fluxo entre os serviços:
Vamos entender quais são os microsserviços que compõem a nossa aplicação e, dentro de cada microsserviço, quais são os seus componentes. Primeiro, nós temos um front-end que foi feito com Angular. Obviamente poderia ter sido feito com qualquer outra tecnologia, mas eu fiz com Angular.

[00:17] E aqui o Angular se comunica diretamente apenas com o API Gateway. Poderíamos ter feito backend for frontend para organizarmos as coisas e garantirmos alguns acessos. Mas enfim, eu utilizei somente o API Gateway para simplificar. Esse API Gateway se comunica com três APIs, ele tem a configuração para redirecionar os requests para três APIs.

[00:38] Uma de marketing, que é a primeira API que consumimos. Então aquela primeira tela de dados pessoais se chama API de Marketing. Lá nós criamos os detalhes com nome e e-mail para criarmos um lead no marketing. Para que isso serve?

[00:56] Imagine que o aluno não completa a matrícula dele, vamos ter o nome e e-mail dele para no futuro mandarmos campanhas, promoções, perguntarmos o motivo por ele não ter terminado a matrícula etc. Então já armazenamos um dado importante desse aluno.

[01:09] Repare que ambos, API e o consumidor de filas (que vamos falar posteriormente), estão no mesmo projeto. Esse projeto foi feito com Node TypeScript e o servidor web é o Express. Já temos uma tecnologia utilizando o microsserviço de marketing.

[01:26] Depois preenchemos as informações financeiras e isso chega no nosso microsserviço de financeiro. Repare que aqui também ambos a tarefa de background (de plano de fundo) e a API estão no mesmo projeto. Isso porque utilizamos a mesma tecnologia e temos um microsserviço feito em PHP e o servidor é Swoole. O Swoole dá algumas facilidades para termos tarefas de plano de fundo. Por isso essa tecnologia foi utilizada.

[01:52] Repare que o banco de dados de marketing é um MongoDB, já o banco de dados do financeiro é um banco relacional. Aqui estou utilizando SQLite só pela simplicidade mesmo, mas poderia ser qualquer outro - Oracle, MySQL, PostgreSQL, SQLServer. Enfim, só estou utilizando SQLite para manter esses dados em memória rapidamente e sem complicar mais.

[02:14] Então, o que acontece? Quando recebemos esses dados na API financeira, ele faz o processamento, recebe os dados e manda para uma tarefa de plano de fundo. Essa tarefa de plano de fundo, no microsserviço financeiro, vai realizar o processamento do pagamento. Aqui poderíamos verificar se os dados do cartão são realmente daquele titular, verificar detalhes de fraude, enviar o pedido para o Gateway de pagamentos e para algum serviço externo etc.

[02:41] Depois que isso é feito, o microsserviço de financeiro manda uma mensagem através da mensageria, através do RabbitMQ. O que esse RabbitMQ é? É basicamente um meio de campo. É um software que fica no meio de várias aplicações recebendo mensagens e essas mensagens são consumidas por outras pessoas. Basicamente, ele é um carteiro.

[03:02] Então o nosso financeiro manda uma mensagem para ele. Vamos ter o microsserviço acadêmico ouvindo essa mensagem e esperando essa carta do microsserviço financeiro. Quando essa carta chega, o nosso microsserviço acadêmico faz o quê? Ele pega o e-mail e o nome desse cliente que o financeiro criou e cria um aluno no banco de dados Postgre. Ele vai criar um aluno e gerar uma senha para ele.

[03:26] E a partir dessa senha gerada ele manda um e-mail para o aluno falando: "a sua matrícula está pronta. É só acessar essa URL que você está pronto para utilizar nosso serviço". Ao mesmo tempo em que isso acontece, o nosso marketing também recebe essa carta. O que o marketing faz? Ele muda o status daquele lead de alguém que está interessado em comprar nossos cursos, para alguém que já virou um de nossos alunos.

[03:49] Dessa forma, essa rotina que é feita no plano de fundo gera uma mensagem que é consumida por dois serviços diferentes. Vamos ver na prática posteriormente como isso é implementado, mas basicamente é assim que isso acontece.

[04:02] Ainda nessa API do microsserviço acadêmico, o que nós temos? O consumidor de filas criou o nosso aluno no banco de dados e a nossa API tem os detalhes de autenticação desse aluno e também tem os detalhes dos cursos que estão disponíveis para ele fazer. Então se marcarmos um curso como assistido, é no banco de dados acadêmico que essa alteração vai ser refletida.

[04:26] Claro que deveríamos mandar essa informação para a mensageria para algum futuro microsserviço de gamificação talvez ler e dar pontos para esse aluno, para o nosso serviço de marketing saber quantos cursos esse aluno já fez e para um outro microsserviço de métricas, talvez de business intelligence.

[04:44] Então, dessa forma funciona a comunicação das nossas APIs, dos nossos microsserviços. Repare que em nenhum momento eu utilizei comunicação direta entre alguma das APIs ou dos microsserviços. Toda a comunicação no nosso processo está sendo feita de forma assíncrona. A única comunicação síncrona é entre o nosso front-end e os nossos microsserviços, ou seja, o frontend espera o retorno das APIs.

[05:10] Agora, as APIs, as tarefas de plano de fundo ou os consumidores de filas, não se importam com a resposta; eles só mandam mensagens. Basicamente, esse é nosso desenho de como está implementado o nosso sistema. No próximo vídeo eu vou te ajudar a trazer esse sistema para o seu computador para você rodar esse sistema, testar e ver se ele realmente está funcionando - ou ver se eu só coloquei um monte de mocks aqui.

