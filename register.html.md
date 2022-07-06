{% extends 'base.html.twig' %}

{% block title %}Register{% endblock %}

{% block body %}
    <div class="container-md col-md-4 col-md-offset-4">
        <h1>Register</h1>

        {{ form_start(registrationForm, { 'attr' : { 'class': 'form-control' } }) }}<br>
        <div class="col-3">
        {{ form_row(registrationForm.email) }}<br>
        {{ form_row(registrationForm.plainPassword, {
            label: 'Password'
        }) }}
        </div><br>
        {{ form_row(registrationForm.agreeTerms) }}<br>
        <button class="btn btn-lg btn-primary" type="submit">
            <a href="{{ path('app_register') }}" class="nav-link" >Register</a>
        </button></p>
{#        <button type="submit" class="btn btn-primary btn-sm">Register</button><br>#}
        <p>Have an account already?
            <button class="btn btn-lg btn-primary" type="submit">
                <a href="/{{ app.user ? 'logout' : 'login' }}" class="nav-link">
                    Sign In</a>
            </button></p>
        {{ form_end(registrationForm) }}
    </div>
    </div>
{% endblock %}
