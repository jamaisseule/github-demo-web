
    <div class="container-md col-md-4 col-md-offset-4">
    <h1 class="h3 mb-3 font-weight-normal">Please sign in</h1>
    <label for="inputEmail">Email</label>
    <input type="email" value="{{ last_username }}" name="email" id="inputEmail" class="form-control" autocomplete="email" required autofocus>
    <br>
        <label for="inputPassword">Password</label>
    <input type="password" name="password" id="inputPassword" class="form-control" autocomplete="current-password" required>

    <input type="hidden" name="_csrf_token"
           value="{{ csrf_token('authenticate') }}"
    >
    <br>

    <button class="btn btn-lg btn-primary" type="submit">
        Sign in
    </button>
        <p>Do not have an account?
            <button class="btn btn-lg btn-primary" type="submit">
                <a href="{{ path('app_register') }}" class="nav-link" >Sign Up</a>
            </button></p>
    </div>
</form>
