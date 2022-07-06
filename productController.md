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
    public function addCart(Product $product, Request $request) //Product $product: id khớp với product, là để quản lý lỡ báo lên 1 id k tồn tại thì k bỏ vô giỏ
    {
        $session = $request->getSession();  //session được sinh ra bởi các request, muốn lấy session  thì phải in check cái request
        $quantity = (int)$request->query->get('quantity'); //challenge debug dung cho bang 0, dùng data validation dấu phẩy phía sau

        //check if cart is empty
        if (!$session->has('cartElements')) { //kt giỏ hàng đang trống thì tạo ra 1 phần tử đầu tiên và điền số lượng vô
            //if it is empty, create an array of pairs (prod Id & quantity) to store first cart element.
            $cartElements = array($product->getId() => $quantity);
            //save the array to the session for the first time.
            $session->set('cartElements', $cartElements); //set array, 1 dãy các cái cặp
        } else { //trường hợp giỏ k trống
            $cartElements = $session->get('cartElements');
            //Add new product after the first time. (would UPDATE new quantity for added product)
            $cartElements = array($product->getId() => $quantity) + $cartElements; // bước thêm  (nhiều hơn 1)
            //Re-save cart Elements back to session again (after update/append new product to shopping cart)
            $session->set('cartElements', $cartElements); //set ngược lại vô các element, truyền ngược vô giỏ hàng
            //lấy tất cả trong hàng ra bỏ cái mới vô rồi bỏ mấy cái cũ vào lại.
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
    public function checkoutCart(Request               $request, //request để trích xuất ra sesion,để lấy ra đc cái giỏ hàng
                                 OrderDetailsRepository $orderDetailRepository, // để ghi xuống db
                                 OrderRepository       $orderRepository,
                                 ProductRepository     $productRepository,
                                 ManagerRegistry       $mr): Response
    {
        $this->denyAccessUnlessGranted('ROLE_USER'); //ngăn cho ông nào chưa đăng nhập là k đc thanh toán
        $entityManager = $mr->getManager(); //trực tiếp làm việc với db
        $session = $request->getSession(); //get a session
        // check if session has elements in cart
        if ($session->has('cartElements') && !empty($session->get('cartElements'))) { //check có rỗng k
            try {
                // start transaction!
                $entityManager->getConnection()->beginTransaction(); //lấy entitymanager ra để quản lý trang retristration
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
                $order->setTotal($total); //để null ở trên để điền total vào đây nếu k phải chạy vòng for hai lần
                $orderRepository->add($order); //nạp vô lại để cập nhập total
                // flush all new changes (all order details and update order's total) to DB
                $entityManager->flush();

                // Commit all changes if all changes are OK
                $entityManager->getConnection()->commit(); //đén đây là k bị lỗi

                // Clean up/Empty the cart data (in session) after all.
                $session->remove('cartElements'); //sau khi mua xong thi remove lun cái giỏ hàng
            } catch (Exception $e) { //bước flush bị lỗi là đến đây
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
