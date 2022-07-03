 /**
     * @Route("/search", name="app_search", methods={"GET"})
     */

    public function index(): Response
    {
        return $this->render('search/searchBar.html.twig', [
            'controller_name' => 'SearchController',
        ]);
    }
    
    
     /**
     * @Route("/search", name="app_search")
     */
    public function searchBar(): Response
    {
        $form = $this->createFormBuilder(null)
//            ->setAction($this->generateUrl('search.handleSearch'))
            ->add('query', TextType::class)
            ->add('search', SubmitType::class, [
                'attr' => [
                    'class' => 'btn btn-primary'
                ]
            ])
            ->getForm();
        return $this->render('search/searchBar.html.twig', [
            'form' => $form->createView()
        ]
