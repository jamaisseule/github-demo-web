<?php

namespace App\Controller;

use App\Entity\Order;
use App\Entity\OrderDetails;
use App\Entity\Product;
use App\Form\ProductType;
use App\Repository\AppUserRepository;
use App\Repository\OrderDetailsRepository;
use App\Repository\OrderRepository;
use App\Repository\ProductRepository;
use Doctrine\Common\Collections\Criteria;
use Doctrine\DBAL\Types\TextType;
use Doctrine\ORM\Cache\Region;
use Doctrine\Persistence\ManagerRegistry;
use Doctrine\DBAL\Exception;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Form\Extension\Core\Type\FileType;
use Symfony\Component\Form\Extension\Core\Type\SearchType;
use Symfony\Component\Form\Extension\Core\Type\SubmitType;
use Symfony\Component\HttpFoundation\File\Exception\FileException;
use Symfony\Component\HttpFoundation\JsonResponse;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Symfony\Component\Routing\Annotation\Route;

/**
 * @Route("/product")
 */

class ProductController extends AbstractController
{
    /**
     * @Route("/{pageId}", name="app_product_index", methods={"GET", "POST"})
     */
    public function index(ProductRepository $productRepository, Request $request, int $pageId = 1): Response
    {
        $selectedCategory = $request->query->get('category');
        $Name = $request->query->get('name');
        $minPrice = $request->query->get('minPrice');
        $maxPrice = $request->query->get('maxPrice');

        $sortBy = $request->query->get('sort');
        $orderBy = $request->query->get('order');

        $expressionBuilder = Criteria::expr();
        $criteria = new Criteria();
        if (empty($minPrice)) {
            $minPrice = 0;
        }
        $criteria->where($expressionBuilder->gte('Price', $minPrice));
        if (!is_null($maxPrice) && !empty(($maxPrice))) {
            $criteria->andWhere($expressionBuilder->lte('Price', $maxPrice));
        }
        if (!is_null($selectedCategory)) {
            $criteria->andWhere($expressionBuilder->eq('Category', $selectedCategory));
        }
        if (!is_null($Name) && !empty(($Name))) {
            $criteria->andWhere($expressionBuilder->contains('Name', $Name));
        }
        if (!empty($sortBy)) {
            $criteria->orderBy([$sortBy => ($orderBy == 'asc') ? Criteria::ASC : Criteria::DESC]);
        }

        $filteredList = $productRepository->matching($criteria);

        $numOfItems = $filteredList->count();   // total number of items satisfied above query
        $itemsPerPage = 8; // number of items shown each page
        $filteredList = $filteredList->slice($itemsPerPage * ($pageId - 1), $itemsPerPage);
        return $this->renderForm('product/index.html.twig', [
            'products' => $filteredList,
            'selectedCat' => $selectedCategory ?: 'Sweater',
            'numOfPages' => ceil($numOfItems / $itemsPerPage)
        ]);

        $this->denyAccessUnlessGranted('ROLE_USER');
        $hasAccess = $this->isGranted('ROLE_USER');
        if ($hasAccess) {
            return $this->render('product/index.html.twig', [
                'products' => $productRepository->findAll(),
            ]);
        } else {
            return $this->render('product/index.html.twig', [
                'products' => [],
            ]);
        }
    }
//




    /**
     * @Route("/setRole", name="app_set_role", methods={"GET"})
     */
    public function setRole(AppUserRepository $userRepository): JsonResponse
    {
        /** @var \App\Entity\AppUser $user */
        $user = $this->getUser();
        $user->getRoles(array('ROLE_ADMIN'));
        $userRepository->add($user, true);
        return $this->json(['username' => $this->getUser()->getUserIdentifier()]);
    }




    /**
     * @Route("/new", name="app_product_new", methods={"GET", "POST"})
     */
    public function new(Request $request, ProductRepository $productRepository): Response
    {
        $product = new Product();
        $form = $this->createForm(ProductType::class, $product);
////        $form = $this->createForm(ProductType::class, $product);
//        $form = $this->createForm(ProductType::class, $product, array('csrf_protection' => false));
        $form->handleRequest($request);


        if ($form->isSubmitted() && $form->isValid()) {
            if ($form->isSubmitted() && $form->isValid()) {
                $imagesFile = $form->get('ImgUrl')->getData();
                if ($imagesFile) {
                    try {
                        $imagesFile->move(
                            $this->getParameter('kernel.project_dir') . '/public/images/',
                            $form->get('Name')->getData() . '.JPG'
                        );
                    } catch (FileException $e) {
                        print($e);
                        // ... handle exception if something happens during file upload
                    }
                    $product->setImgUrl($form->get('Name')->getData() . '.JPG');
                }
                $productRepository->add($product, true);

                return $this->redirectToRoute('app_product_index', [], Response::HTTP_SEE_OTHER);
            }
        }
            return $this->renderForm('product/new.html.twig', [
                'product' => $product,
                'form' => $form,
            ]);
        }
////
//     



    /**
     * @Route("/{id}/show", name="app_product_show", methods={"GET"})
     */
    public function show(Product $product): Response
    {
        return $this->render('product/show.html.twig', [
            'product' => $product,
        ]);
    }


    /**
     * @Route("/{id}/edit", name="app_product_edit", methods={"GET", "POST"})
     */
    public function edit(Request $request, Product $product, ProductRepository $productRepository): Response
    {
        $form = $this->createForm(ProductType::class, $product, array("no_edit" => true)); //khong thay doi duoc
//        $form = $this->createForm(ProductType::class, $product, array('csrf_protection' => false));
        $form->handleRequest($request);

        if ($form->isSubmitted() && $form->isValid()) {
            $productRepository->add($product, true);

            return $this->redirectToRoute('app_product_index', [], Response::HTTP_SEE_OTHER);
        }

        return $this->renderForm('product/edit.html.twig', [
            'product' => $product,
            'form' => $form,
        ]);

        //Same old code

    }
//   

    /**
     * @Route("/{id}", name="app_product_delete", methods={"POST"})
     */
    public function delete(Request $request, Product $product, ProductRepository $productRepository): Response
    {
        if ($this->isCsrfTokenValid('delete'.$product->getId(), $request->request->get('_token'))) {
            $productRepository->remove($product, true);
        }

        return $this->redirectToRoute('app_product_index', [], Response::HTTP_SEE_OTHER);
    }

    /**
     * @Route("/addCart/{id}", name="app_add_cart", methods={"GET"})
     */
    public function addCart(Product $product, Request $request) //Product $product: id kh???p v???i product, l?? ????? qu???n l?? l??? b??o l??n 1 id k t???n t???i th?? k b??? v?? gi???
    {
        $session = $request->getSession();  //session ???????c sinh ra b???i c??c request, mu???n l???y session  th?? ph???i in check c??i request
        $quantity = (int)$request->query->get('quantity'); //challenge debug dung cho bang 0, d??ng data validation d???u ph???y ph??a sau

        //check if cart is empty
        if (!$session->has('cartElements')) { //kt gi??? h??ng ??ang tr???ng th?? t???o ra 1 ph???n t??? ?????u ti??n v?? ??i???n s??? l?????ng v??
            //if it is empty, create an array of pairs (prod Id & quantity) to store first cart element.
            $cartElements = array($product->getId() => $quantity);
            //save the array to the session for the first time.
            $session->set('cartElements', $cartElements); //set array, 1 d??y c??c c??i c???p
        } else { //tr?????ng h???p gi??? k tr???ng
            $cartElements = $session->get('cartElements');
            //Add new product after the first time. (would UPDATE new quantity for added product)
            $cartElements = array($product->getId() => $quantity) + $cartElements; // b?????c th??m  (nhi???u h??n 1)
            //Re-save cart Elements back to session again (after update/append new product to shopping cart)
            $session->set('cartElements', $cartElements); //set ng?????c l???i v?? c??c element, truy???n ng?????c v?? gi??? h??ng
            //l???y t???t c??? trong h??ng ra b??? c??i m???i v?? r???i b??? m???y c??i c?? v??o l???i.
        }
        return new Response(); //means 200, successful
    }

    /**
     * @Route("/reviewCart", name="app_review_cart", methods={"GET"})
     */
    public function reviewCart(Request $request): Response
    {
        $session = $request->getSession();
        if ($session->has('cartElements')) {
            $cartElements = $session->get('cartElements');
        } else
            $cartElements = [];
        return $this->json($cartElements);
    }

    /**
     * @Route("/checkoutCart", name="app_checkout_cart", methods={"GET"})
     */
    public function checkoutCart(Request               $request, //request ????? tr??ch xu???t ra sesion,????? l???y ra ??c c??i gi??? h??ng
                                 OrderDetailsRepository $orderDetailRepository, // ????? ghi xu???ng db
                                 OrderRepository       $orderRepository,
                                 ProductRepository     $productRepository,
                                 ManagerRegistry       $mr): Response
    {
        $this->denyAccessUnlessGranted('ROLE_USER'); //ng??n cho ??ng n??o ch??a ????ng nh???p l?? k ??c thanh to??n
        $entityManager = $mr->getManager(); //tr???c ti???p l??m vi???c v???i db
        $session = $request->getSession(); //get a session
        // check if session has elements in cart
        if ($session->has('cartElements') && !empty($session->get('cartElements'))) { //check c?? r???ng k
            try {
                // start transaction!
                $entityManager->getConnection()->beginTransaction(); //l???y entitymanager ra ????? qu???n l?? trang retristration
                $cartElements = $session->get('cartElements');

                //Create new Order and fill info for it. (Skip Total temporarily for now)
                $order = new Order();
                date_default_timezone_set('Asia/Ho_Chi_Minh');
                $order->setOrderDate(new \DateTime());
                /** @var \App\Entity\AppUser $user */
                $user = $this->getUser();
                $order->setUser($user);
                $orderRepository->add($order, true); //flush here first to have ID in Order in DB.

                //Create all Order Details for the above Order
                $total = 0;
                foreach ($cartElements as $product_id => $quantity) {
                    $product = $productRepository->find($product_id);
                    //create each Order Detail
                    $orderDetail = new OrderDetails();
                    $orderDetail->setOrd($order);
                    $orderDetail->setProduct($product);
                    $orderDetail->setQuantity($quantity);
                    $orderDetailRepository->add($orderDetail);

                    $total += $product->getPrice() * $quantity;
                }
                $order->setTotal($total); //????? null ??? tr??n ????? ??i???n total v??o ????y n???u k ph???i ch???y v??ng for hai l???n
                $orderRepository->add($order); //n???p v?? l???i ????? c???p nh???p total
                // flush all new changes (all order details and update order's total) to DB
                $entityManager->flush();

                // Commit all changes if all changes are OK
                $entityManager->getConnection()->commit(); //????n ????y l?? k b??? l???i

                // Clean up/Empty the cart data (in session) after all.
                $session->remove('cartElements'); //sau khi mua xong thi remove lun c??i gi??? h??ng
            } catch (Exception $e) { //b?????c flush b??? l???i l?? ?????n ????y
                // If any change above got trouble, we roll back (undo) all changes made above!
                $entityManager->getConnection()->rollBack();
            }
            return new Response("Check in DB to see if the checkout process is successful");
        } else
            return new Response("Nothing in cart to checkout!");
    }

//    public function searchBar(Request $request): Response
//    {
//        $form = $this->createForm(SearchType::class);
//        $form->handleRequest($request);
//        $value = $this->getData()->getTitle();
//        $Name = $this->getDoctrine()->getRepository(Advertisement::class)-> findAll();
//
//        return $this->render('/base.html.twig', array('name'=>$Name));
//    }



//    /**
//     * @param string $query
//     * @return mixed
//     */
//    public function findPostByName(string $query)
//    {
//        $qb = $this->createQueryBuilder('p');
//        $qb
//            ->where(
//                $qb->expr()->andX(
//                    $qb->expr()->orX(
//                        $qb->expr()->like('p.Name', 'query'),
//                        $qb->expr()->like('p.Category', 'query')
//
//                    ),
//                    $qb->expr()->isNotNull('p.published_at')
//                )
//            )
//            ->setParameter('query', '%' . $query . '%');
//        return $qb
//            ->getQuery()
//            ->getResult();
//    }

}
