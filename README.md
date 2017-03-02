# GAN-Generative-Adversarial-Networks-Parte-I

![output](data-h.jpg) ![output](deeplearningbrasil.png)

Introdu��o 

Arquiteturas de redes neurais Generativas (ou geradoras) possuem como principal caracter�stica a habilidade de "aprender' a criar dados artificiais que s�o semelhantes a dados reais. A premissa b�sica �: "O que voc� n�o consegue criar, n�o consegue entender". Entretanto, seu entendimento e uso ainda n�o se apresentam como uma tarefa f�cil. Neste post pretendemos descrever a formula��o da GAN e fornecer um breve exemplo com c�digo no TensorFlow para um problema relativamente simples e de f�cil compreens�o.

## Ambiente de codifica��o

O c�digo deste exemplo foi testado no Linux Ubuntu 14 

* Python 2.7
* tensorflow (1.0) - Para vers�es < 1, descomente a linha 90 e comente a linha 92.
* matplotlib
* numpy
* scipy
* seaborn

Para executar:
* python2 gan.py --num-steps 1000 --minibatch True

## Compreendendo a Arquitetura

A ideia da arquitetura GAN foi introduzida pela primeira vez em 2014 por um grupo de pesquisadores da Universidade de Montreal liderado por Ian Goodfellow (atualmente na OpenAI). A arquitetura b�sica � composta por duas redes neurais concorrentes conforme ilustrado abaixo:


![output](arquitetura_gan.jpg)

Uma rede neural (chamada de geradora) toma "ru�do" como entrada e gera dados em sua sa�da. A outra rede neural (chamada de discriminadora) recebe dados tanto da rede geradora quanto de dados reais de treinamento. A rede discriminadora deve ser capaz de distinguir entre as duas fontes, rotulando a informa��o como real ou sint�tica (falsa). Estas duas redes praticam um "jogo" cont�nuo, onde o gerador est� aprendendo a produzir dados cada vez mais realistas, e a discriminadora est� aprendendo a distinguir dados sint�ticos de dados reais. Essas duas redes s�o treinadas simultaneamente com o objetivo de que a competi��o conduza a situa��o de que os dados sint�ticos se tornem  indistingu�veis dos dados reais.

## Explica��o modo "Easy"

Vamos utilizar o epis�dio No Weenies Allowed (N�o adimitimos piralhos - tradu��o livre) do desenho Bob Esponja para explicar como funciona as GAN's. A fun��o do seguran�a do clube (nosso discriminador GAN) � permitir a entrada somente de "caras fortes" e negar a entrada de "piralhos" curiosos. 


![](http://vignette3.wikia.nocookie.net/spongebob/images/0/0c/Noweenie.jpg/revision/latest?cb=20121215100522).

O seguran�a (D - discriminador) barra Bob Esponja (G - Gerador) por consider�-lo fracote. De fato Bob Esponja (G) � um imitador. Ele precisa se asemelhar o m�ximo poss�vel para enganar o seguran�a. 

![](https://thumbs.gear3rd.net/55,e595b57d79fbab.jpg)

Bob esponja, tenta de todas as formas se disfar�ar, alterando a sua apar�ncia de modo a ficar similar aos indiv�duos que adentram o clube para tentar "enganar" o seguran�a. O seguran�a em um primeiro momento observa aspectos �bvios que discrimine as duas categorias, cabe a Bob Esponja (G) imitar esses aspectos �bvios para n�o ser reconhecido. Uma vez que o discriminador (D) definiu algum aspecto como importante (a for�a, por exemplo), o gerador (G) usar� essa informa��o para tentar "enganar" o discriminador.

![](bobesponja.png)

O epis�dio segue com Bob Esponja (G) tentando exibir/imitar caracter�sticas que tentam enganar o seguran�a (D). Fazendo uma analogia com as imagens do desenho, � poss�vel ilustrar na figura abaixo o funcionamento das redes. O geradaor ter� n�meros rand�micos como entrada e produzir� imagens que ser�o avaliadas por uma rede de discrimina��o que determinar� se a imagem � falsa ou real.

![output](https://cdn-images-1.medium.com/max/1000/1*39Nnni_nhPDaLu9AnTLoWw.png)




## Explica��o modo "Hard"

Se voc� n�o tem medo de matem�tica, vamos direto ao ponto. As redes GAN podem ser resumidas pela equa��o abaixo:

![output](equacao_gan.png)

Mas calma... se n�o entendeu, vamos compreend�-la.

O gerador (G) e o driscriminador (D) geralmente s�o redes neurais do tipo "feedforward". A primeira parcela ![output](https://github.com/tensorflow/magenta/raw/master/magenta/reviews/assets/gan/image06.png) expressa que o discriminador (D) ir� buscar os par�metros da rede que a levem a discriminar entre dados reais e sint�ticos. J� na segunda parcela ![output](https://github.com/tensorflow/magenta/raw/master/magenta/reviews/assets/gan/image13.png), o gerador (G) ter� como objetivo gerar sa�das que o discriminador (D) considere como real. A entrada z, � um vetor de ru�dos de uma distribui��o ![output](https://github.com/tensorflow/magenta/raw/master/magenta/reviews/assets/gan/image10.png). O discriminador recebe como entrada um conjunto de dados real (x) ou gerado (G(z)), e produz como sa�da uma probabilidade da entrada ser real P(x).

Ambas as partes das equa��es podem ser otimizadas de forma independente com m�todos baseados em gradientes. Primeiro, fa�a um passo para maximizar o discriminador (D) ![output](https://github.com/tensorflow/magenta/raw/master/magenta/reviews/assets/gan/image08.png), depois fa�a um passo para otimizar o gerador (G) de modo a minimizar ![output](https://github.com/tensorflow/magenta/raw/master/magenta/reviews/assets/gan/image12.png) a separa��o encontrada pelo discriminador (D). Em termos pr�ticos, estamos seguindo uma regra min-max da teoria dos jogos, uma rede joga contra a outra. 


## Exemplo pr�tico:

Para entender melhor como tudo isso funciona, usaremos um GAN para resolver um problema did�tio usando o framework TensorFlow. O desafio � um "toy problem". A rede neural dever� ser capaz de aproximar uma distribui��o Gaussiana unidimensional. O c�digo-fonte completo est� dispon�vel em nosso Github ( https://github.com/AYLIEN/gan-intro ). Neste texto vamos focar nas partes mais importantes do c�digo.

Primeiro criamos o conjunto de dados "real", um curva gaussiana simples com m�dia igual a 4 (quatro) e desvio padr�o de 0,5. Para isso, implementamos a fun��o abaixo na linguagem python, que retorna um determinado n�mero de amostras da distribui��o de dados (gaussiana).

```python
class distribuicaoDados(object):
    def __init__(self):
        self.media = 4
        self.desviopadrao = 0.5
        
    def amostras(self, N):
        amostras = np.random.normal(self.memdia, self.desviopadrao, N)
        amostras.sort()
```


Definiremos tamb�m os dados de entrada para a rede neural geradora, utilizando a mesma fun��o de distribui��o de dados, por�m, perturbando os dados com ru�do artificial. 


```python
class distribuicaoDados(object):
    def __init__(self, range):
        self.range = range

    def amostras(self, N): 
       return np.linspace(-self.range, self.range, N) + np.random.random(N)  0.01
```
   
Nosso gerador, por simplicidade deste exemplo, n�o ser� uma rede neural propriamente dita. Ser� uma fun��o linear, seguida por uma n�o-linear e por fim uma nova fun��o linear, conforme codificado abaixo.

```python
def gerador(input, hidden_size):
    passo1 = linear(input,hidden_size,'g0')
    passo2 = tf.nn.softplus(passo1)
    passo3 = linear(passo2, 1, 'g1')
    return passo3            `
```
    
Neste caso, � importante que o discriminador seja mais "poderoso" do que o gerador, ou ent�o, corremos o risco de que ele n�o tenha capacidade suficiente para distinguir entre os dados sint�ticos e os reais. Logo, vamos definir uma rede neural com fun��es tanh (n�o-lineares) em todas as camadas, exceto na final, que conter� uma fun��o sigm�ide que ir� gerar valores entre 0 e 1 que podem ser interpretados como o resultado de probabilidade de ser verdadeiro (valor 1) ou falso (valor 0).

```python
def discriminador(input, h_dim, minibatch_layer=True):
    passo1_h0 = linear(input,h_dim*2,'d0')
    passo2_h0 = tf.tanh(passo1_h0)
    passo1_h1 = linear(passo2_h0,h_dim*2,'d1')
    passo2_h1 = tf.tanh(passo1_h1)
```
    
Agora podemos colocar as fun��es j� desenvolvidas em um "TensorFlow graph" e tamb�m definir uma fun��o de "perda" , geralmente chamada de "loss" nos artigos cient�ficos (rela��o entre o que se espera e o que foi obtido), tendo como alvo a situa��o em que o discriminador n�o seja capaz de distinguir entre o dado sint�tico e o verdadeiro.

```python
        with tf.variable_scope('Gen'):
            self.z = tf.placeholder(tf.float32, shape=(self.batch_size, 1))
            self.G = gerador(self.z, self.mlp_hidden_size)

        with tf.variable_scope('Disc') as scope:
            self.x = tf.placeholder(tf.float32, shape=(self.batch_size, 1))
            self.D1 = discriminador(self.x, self.mlp_hidden_size, self.minibatch)
            scope.reuse_variables()
            #cria uma segunda rede de discriminacao, pois no tensorflow nao seria 
            #possivel uma rede com duas entradas diferentes
            self.D2 = discriminador(self.G, self.mlp_hidden_size, self.minibatch)
            
        self.loss_d = tf.reduce_mean(-tf.log(self.D1) - tf.log(1 - self.D2))
        self.loss_g = tf.reduce_mean(-tf.log(self.D2))
```

Criamos fun��es de otimiza��o para cada rede usando o crit�rio de GradientDescentOptimizer do TensorFlow com taxa de aprendizado com decaimento exponencial. Note que existem alguns conceitos envolvidos aqui. O completo entendimento e compreens�o dos par�metros de otimiza��o requer alguma experi�ncia pr�via com otimiza��o e de redes neurais artificiais.

```python
    def otimizador(erro, var_list, taxa_aprendizado_inicial):
        decaimento = 0.95
        qtd_passos_decaimento = 150
        batch = tf.Variable(0)
        learning_rate = tf.train.exponential_decay(
            taxa_aprendizado_inicial,
            batch,
            qtd_passos_decaimento,
            decaimento,
            staircase=True
        )
        otimizador = tf.train.GradientDescentOptimizer(learning_rate).minimize(
            erro,
            global_step=batch,
            var_list=var_list
        )
        return otimizador
```
    
Para treinar a GAN, obt�m-se pontos da distribui��o de dados e da distribui��o sint�tica e alternamos entre otimizar os par�metros do discriminador e do gerador.

 ```python
        def train(self):
        with tf.Session() as session:
            tf.global_variables_initializer().run()

        for step in xrange(self.num_steps):
            # atualiza o discriminador
            x = self.data.sample(self.batch_size)
            z = self.gen.sample(self.batch_size)
            loss_d, _ = session.run([self.loss_d, self.opt_d], {
                self.x: np.reshape(x, (self.batch_size, 1)),
                self.z: np.reshape(z, (self.batch_size, 1))
            })

            # atualiza o gerador
            z = self.gen.sample(self.batch_size)
            loss_g, _ = session.run([self.loss_g, self.opt_g], {
                self.z: np.reshape(z, (self.batch_size, 1))
            })
```

O v�deo abaixo mostra a sequ�ncia de �pocas de treinamento em que a rede geradora tenta aprender a imitar a distribui��o real dos dados durante o treinamento. � poss�vel notar que no in�cio, a rede geradora produz uma distribui��o muito distinta da real. Com o treinamento a rede consegue gerar uma distribui��o similar a real.

Video:
<a href="http://www.youtube.com/watch?feature=player_embedded&v=Z-9DWDnT9lA
" target="_blank"><img src="http://img.youtube.com/vi/Z-9DWDnT9lA/0.jpg" 
alt="Demonstra��o treinamento GAN" width="340" height="280" border="1" /></a>


Talvez um dos fatos mais instigantes das redes GAN's esteja no fato de que as redes competem entre si pelo melhor resultado, por�m, s� o conseguir�o em caso de equil�brio entre o resultado de ambas. A figura abaixo mostra a evolu��o da fun��o de erro ou perda nas primeiras gera��es. A medida que uma rede melhora o seu resultado o da outra piora. Este "ponto de equil�brio" atualmente � o maior desafio em aberto no processo de treinamento deste tipo de rede neural.

![output](treinamento-gan.png)

## Considera��es Finais

Redes GAN apresentam-se como uma ideia bastante interessante, dando-nos uma nova abordagem no processo de aprendizagem. As aplica��es  bem sucedidas de GANs ainda est�o em problemas de vis�o computacional. O crit�rio de parada do processo de treinamento ainda � um problema em aberto. Em aplica��es de vis�o computacional podemos olhar para as imagens geradas e fazer um julgamento se a solu��o � satisfat�ria, contudo, em outros tipos de aplica��es, esse tipo de observa��o n�o � trivial.

Apesar destas limita��es e dificuldades, a comunidade cient�fica em geral est� bastante animada em raz�o da premissa b�sica de que, conseguimos entender aquilo que conseguimos gerar, logo, os primeiros passos foram dados para que em breve tenhamos redes neurais artificiais que de fato consigam aprender "sozinhas" a partir de uma base de conhecimento. Nessa situa��o, podemos imaginar por exemplo, redes neurais que escrevem textos a partir de um simples dep�sito de colet�neas de artigos em sua base de dados ou que componham m�sicas a partir de uma biblioteca digital qualquer. Por fim, diria que a GAN � um touro arisco em processo de domestica��o, quem conseguir o feito certamente levar� o grande pr�mio. Fa�am suas apostas.