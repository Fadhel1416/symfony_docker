# symfony project with docker features

#install docker and docker-compose

#clone the project

#build the docker images by runiing this command :run docker-compose up -d --build

#run docker exec -it php bash
you will be in the php container

#run symfony new  .
composer req --dev maker ormfixtures fakerphp/faker
composer req doctrine twig
cp .env .env.local

add this in .env.local DATABASE_URL="mysql://root:secret@database:3306/symfony_docker?serverVersion=8.0"
symfony console make:entity Quote
add field to this entity
add this in entity as constratctor 
public function __construct($quote, $historian, $year) {
    $this->quote = $quote;
    $this->historian = $historian;
    $this->year = $year;
}
symfony console make:migration
symfony console doctrine:migrations:migrate
docker-compose exec database /bin/bash
mysql -u root -p symfony_docker
in php container run symfony console make:fixture QuoteFixture
add this code to QuoteFixture
<?php

namespace App\DataFixtures;

use App\Entity\Quote;
use Doctrine\Bundle\FixturesBundle\Fixture;
use Doctrine\Persistence\ObjectManager;
use Faker\Factory;

class QuoteFixture extends Fixture {

    private $faker;

    public function __construct() {

        $this->faker = Factory::create();
    }

    public function load(ObjectManager $manager) {

        for ($i = 0; $i < 50; $i++) {
            $manager->persist($this->getQuote());
        }
        $manager->flush();
    }

    private function getQuote() {

        return new Quote(
            $this->faker->sentence(10),
            $this->faker->name(),
            $this->faker->year()
        );
    }
}
run symfony console doctrine:fixtures:load
symfony console make:controller QuoteController
<?php

namespace App\Controller;

use App\Repository\QuoteRepository;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\\Annotation\Route;

class QuoteController extends AbstractController {
    #[Route('/', name: 'index')]
    public function index(
        QuoteRepository $quoteRepository
    )
    : Response {

        return $this->render(
            'quote/index.html.twig',
            [
                'quotes' => $quoteRepository->findAll(),
            ]
        );
    }
}
