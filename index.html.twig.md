{% extends 'base.html.twig' %}

{% block title %}Product index{% endblock %}

{% block body %}
<div class="container-md">

    <div class="input-group">

    <div class="row">

        <div class="col-3">
            <form action={{ path('app_product_index') }} method="get">
                    <input type="string" name="name" class="form-control rounded mb-4"
                           placeholder=Search aria-label="Search"
                           aria-describedby="search-addon"/>

                <h6>Category:</h6>
{#                    <select name="form-check">#}
{#                        <option value="serum" {{ (selectedCat=="serum") ? 'selected' }}> serum</option>#}
{#                        <option value="Snack" {{ (selectedCat=="Snack") ? 'selected' }}> Snack</option>#}
{#                        <option value="Electronics" {{ (selectedCat=="Electronics") ? 'selected' }} >Electronics</option>#}
{#                    </select><br><br>#}

                    <input type="checkbox" value="Sweater" {{ (selectedCat=="Sweater") ? 'selected' }}>
                    <label>Sweater</label><br/>
                    <input type="checkbox" value="Blazer" {{ (selectedCat=="Blazer") ? 'selected' }}>
                    <label>Blazer</label><br/>
                    <input type="checkbox" value="Pant" {{ (selectedCat=="Pant") ? 'selected' }}>
                    <label>Pant</label><br/>
                    <input type="checkbox" value="Dress" {{ (selectedCat=="Dress") ? 'selected' }}>
                    <label>Dress</label><br/>
                    <input type="checkbox" value="Shoes" {{ (selectedCat=="Shoes") ? 'selected' }}>
                    <label>Shoes</label><br/>

                <br>
                <h6>Min Price:</h6> <input type="number" name="minPrice"><br>
                <h6>Max Price:</h6> <input type="number" name="maxPrice"><br><br>
                    <input type="submit" value="Filter" class="btn btn-primary"> |
                    <a href="{{ path('app_product_index') }}"
                       class="btn btn-primary">Reset</a>
            </form>
        </div>
        <div class="col-9">
            <div class="container">
                <div class="row">
                    <div class="col-6"><h4>Product Listing</h4></div>
                    <div class="col-3 text-end"><p>Price: </p></div>
                         <div class="col-3">
                            <a href="{{ path(app.request.attributes.get('_route'),
                                app.request.query.all|merge({'sort': 'Price', 'order': 'desc'})) }}"
                               class="btn btn-primary btn-sm">Hi->Low</a>
                            <a href="{{ path(app.request.attributes.get('_route'),
                                app.request.query.all|merge({'sort': 'Price', 'order': 'asc'})) }}"
                               class="btn btn-primary btn-sm">Low->Hi</a>
                        </div>
                    </div>
                <div class="row row-cols-1 row-cols-md-4 g-4">
                    {% for product in products %}
                        <div class="col">
                            <div class="card h-100">
{#                                <img class="card-img-top" src="{{ asset('images/logo.jpg') }}">#}
                                <img class="card-img-top" src="{{ asset('images/{{ product.ImgUrl }}.jpg') }}">
                                <div class="card-body">
                                    <h5 class="card-title">{{ product.Name }}</h5>
                                    <h6 class="card-subtitle">Category: {{ product.category }}</h6>
                                    <p>Price: {{ product.price }}</p>
                                </div>
                                <div class="card-footer">
                                    <a href="{{ path('app_product_show', {'id': product.id}) }}"
                                       class="btn btn-primary">Show</a>
                                </div>
                            </div>
                        </div>
                    {% endfor %}
                </div>
                <br>
                <div class="row mt-2">
                    <div style="display:flex;text-align:center;justify-content:center">
                        <nav aria-label="Page navigation">
                            <ul class="pagination">
                                {% for i in range(1, numOfPages) %}
                                    {% set style = app.request.get('pageId')==i ? "active" : "" %}
                                    <li class="page-item {{ style }}">
                                        <a class="page-link"
                                           href={{ path(app.request.attributes.get('_route'),
                                            app.request.query.all|merge({'pageId': i})) }}>{{ i }}</a>
                                    </li>
                                {% endfor %}
                            </ul>
                        </nav>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
</div>
{% endblock %}
