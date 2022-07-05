# Documentation

## Installation du Package

```bash

    composer req enqueue/rdkafka "^0.10.15"

```

## Emission des donnée

- Création d'un dossier `src/Services` si ça n'existe pas

- Création d'une classe `App\Services\Receive` dans le ficher `src/Services/Receive.php`

- inserer ce code

```php

    <?php

        namespace App\Services;

        use Doctrine\ORM\EntityManagerInterface;
        use Symfony\Component\DependencyInjection\ContainerInterface;

        use Symfony\Contracts\HttpClient\HttpClientInterface;
        use RdKafka\Conf;
        use RdKafka\KafkaConsumer;

        use Enqueue\RdKafka\RdKafkaConnectionFactory;
        use Enqueue\RdKafka\RdKafkaMessage;
        use Enqueue\RdKafka\RdKafkaContext;
        use Enqueue\RdKafka\RdKafkaConsumer;



        /**
         * Perment de créer un un kafka connect
         */
        class Receive {

            /**
             * @var EntityManagerInterface
             */
            private $em;
            /**
             * @var ContainerInterface
             */
            private $container;
            /**
             * @var RdKafkaContext
             */
            private $context;

            /**
             * 
             */
            public function __construct(
                EntityManagerInterface $em, 
                ContainerInterface $container,
            ) {
                $this->em = $em;
                $this->container = $container;
            }

            /**
             * @return RdKafkaContext
             */
            public function getContext()
            {
                if (is_null($this->context)) {

                    // Le parametre qui a l'adresse ip ou le nom de domain du serveur
                    $host = $this->container->getParameter("kafka.hostname");

                    $connectionFactory = new RdKafkaConnectionFactory([
                        'global' => [
                            'group.id' => uniqid('', true),
                            'metadata.broker.list' => $host,
                            'enable.auto.commit' => 'false',
                        ],
                        'topic' => [
                            'auto.offset.reset' => 'beginning',
                        ],
                    ]);
            
                    $context = $connectionFactory->createContext();
            
                    $this->context = $context;
                }

                return $this->context;

            }
            /**
             * @return RdKafkaConsumer
             */
            public function getConcumer()
            {
                $context = $this->getContext();

                // Le parametre qui a le nom du sujet
                $topicName = $this->container->getParameter("kafka.topic.name");

                //dump($topicName);

                $fooQueue = $context->createTopic($topicName);

                $consumer = $context->createConsumer($fooQueue);

                return $consumer;
            }

            /**
             * 
             */
            public function __invoke(array $data = [])
            {
                
                $consumer = $this->getConcumer();


                $message = $consumer->receive();
                
                
                $consumer->acknowledge($message);

                return $message->getBody();
            }


```