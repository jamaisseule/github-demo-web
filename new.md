# github-demo-web
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
