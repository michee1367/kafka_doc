# Documentation

## Installation du Package

```bash

    composer req enqueue/rdkafka "^0.10.15"

```

## Emission des donnée

- Création d'un dossier `src/Services` si ça n'existe pas

- Création d'une classe `App\Services\Emit` dans le ficher `src/Services/Emit.php`

- inserer ce code

```php

    <?php

        namespace App\Services;
        
        use Enqueue\RdKafka\RdKafkaConnectionFactory;
        use Symfony\Component\DependencyInjection\ContainerInterface;
        use Mink67\KafkaConnect\Constant;
        use Symfony\Component\Serializer\Normalizer\NormalizerInterface;
        use Doctrine\Common\Util\ClassUtils;
        use Enqueue\RdKafka\RdKafkaContext;
        use Enqueue\RdKafka\RdKafkaProducer;


        /**
         * Perment de créer un un kafka connect
         */
        class Emit {

            /**
             * @var ContainerInterface
             */
            private $container;
            /**
             * @var NormalizerInterface
             */
            private $normalizer;
            /**
             * @var RdKafkaContext
             */
            private $context;
            /**
             * @var RdKafkaProducer
             */
            private $productor;

            /**
             * 
             */
            public function __construct(
                ContainerInterface $container, 
                NormalizerInterface $normalizer,
            ) {
                $this->container = $container;
                $this->normalizer = $normalizer;
            }
            /**
             * @return RdKafkaContext
             */
            public function getContext()
            {
                if (is_null($this->context)) {
                    

                    $groupId = uniqid('', true);

                    // Le parametre qui a l'adresse ip ou le nom de domain du serveur
                    $host = $this->container->getParameter("kafka.hostname");

                    $autoOffsetReset = "beginning";

                    
                    $connectionFactory = new RdKafkaConnectionFactory([
                        'global' => [
                            'group.id' => $groupId,
                            'metadata.broker.list' => $host,
                            'enable.auto.commit' => 'false',
                        ],
                        'topic' => [
                            'auto.offset.reset' => $autoOffsetReset,
                        ],
                    ]);
                    
                    $context = $connectionFactory->createContext();
                    $this->context = $context;
                }

                return $this->context;
                
            }

            /**
             * @return RdKafkaProducer
             */
            public function getProducer()
            {
                if (is_null($this->productor)) {

                    $context = $this->getContext();
            
                    $productor = $context->createProducer();

                    $this->productor = $productor;
                    
                }

                return $this->productor;
            }


            /**
             * 
             */
            public function __invoke($entity)
            {
                //define('RD_KAFKA_VERSION', 16908799);

                if (is_null($action)) {
                    $action = Constant::CREATE_ACTION;
                }

                // Le parametre qui a le groupe de normalisation
                $groups = $this->container->getParameter("kafka.normalization.group");

                $dataArr = $this->normalizer
                                ->normalize(
                                    $entity, 
                                    null,
                                    [
                                        'groups' => $groups,
                                    ]
                            );
                
                $messageArr = [
                            'data' => $dataArr,
                            'metaData' => [
                                'groups' => $groups
                            ],
                        ];

                $messageStr = json_encode($messageArr);
                $messageBase64 = base64_encode($messageStr);

                
                // Le parametre qui a le nom du sujet
                $topicName = $this->container->getParameter("kafka.topic.name");
                
                //$context = $connectionFactory->createContext();
                $context = $this->getContext();
                
                $message = $context->createMessage($messageBase64);
                
                $topic = $context->createTopic($topicName);
                
                $this->getProducer()->send($topic, $message);

                //dd($topic);

            }
        }


```