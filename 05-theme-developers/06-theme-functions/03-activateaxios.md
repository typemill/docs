# activateAxios()

The Twig-function `{{ assets.activateAxios() }}` will add the axios-library for http-calls to your theme and initialise it with the base-url and the variably `tmaxios`. 

## Example Usage

````
<body>
  ...
  {{ content }}
  ...

  {% block javascripts %}

   {{ assets.activateAxios() }}

   <script>
            tmaxios.post('/api/v1/myendpoint',{
                'something': 'valid',
            })
            .then(function (response)
            {
            })
            .catch(function (error)
            {
            });
   <script>

  {% endblock %}   
</body>
````

