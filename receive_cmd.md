# Documentation

## Create Command

```text
    le nom de la commande sera `kafka:consumer`
```

```bash

    php bin/console make:command

```

## Modification de la commmande


- Acceder à la classe `App\Command\KafkaReceiveExCommand` dans le ficher `src/Command/KafkaReceiveExCommand.php`

- Ajouter le service du consommateur

```php

    <?php

        ...
        use App\Services\Receive as ServicesReceive;
        use Symfony\Component\Serializer\Normalizer\DenormalizerInterface; 
        use Doctrine\Common\Util\ClassUtils;
        use Symfony\Component\Serializer\Normalizer\AbstractNormalizer;
        ...

        public function __construct(
            ServicesReceive $receive,
            EntityManagerInterface $em,
            DenormalizerInterface $denormalizer,
        ) 
        {
            $this->receive = $receive;
            $this->em = $em;
            $this->denormalizer = $denormalizer;

            parent::__construct();
        }
```

- Modification de la methode execute

```php


    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $io = new SymfonyStyle($input, $output);

        do {
            $receive = $this->receive;
            
            $messageBase64 = $receive();
            $messageStr = base64_decode($messageBase64);

            $messageArr = \json_decode($messageStr, true);

            $entity = new Entity;
            
            // Le parametre qui a le groupe de normalisation
            $groups = $this->container->getParameter("kafka.normalization.group");

            $entity = $this->denormalizer
                            ->denormalize(
                                $data,
                                ClassUtils::getClass($entity),
                                null,
                                [
                                    'groups' => $groups,
                                    AbstractNormalizer::OBJECT_TO_POPULATE => $entity
                                ]
                        );

            
            $this->em->persist($entity);
            $this->em->flush();
            
        } while (true);

        return Command::SUCCESS;
    }

```

- Executer la commande

```bash
    php bin/console kafka:consumer
```