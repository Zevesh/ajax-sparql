## &lt;ajax-sparql&gt;
`ajax-sparql` is a simple component that automatizes the creation of a request for an SPARQL site just by taking an SPARQL query. That allows the code to be more readable, because you can see the original query instead of the resulting url. It can also be used to create queries based on the content of a variable, being not necessary to know the data at coding time. 

When creating an instance of this component, we need to know the URL of the site where we will execute the query and the query itself. Sometimes we also need the graph inside the site, but usually it is optional.


### Declaration of the component.
When creating an element of `ajax-sparql` the properties `url` and `query` are always required, whereas `graph` is optional. There is another point to have in mind. This component have no `on-response` attribute, so the treatment of the data must be done manually. To achieve this, first we have to give it an `id` to simplify the later access. 

Example:
```
<ajax-sparql 
id="request"
url="http://opendata.caceres.es/sparql/"
graph="http://opendata.caceres.es/recurso/salud/centros/"
query="select ?name ?latitude ?longitude where{
  ?URI a schema:MedicalOrganization.
  ?URI rdfs:label ?name.
  ?URI geo:lat ?latitude.
  ?URI geo:long ?longitude. 
  }">
</ajax-sparql>
```

The `query` property admits various formats: in a single line, in multiple lines, with keywords in upper or lower case... Choose wherever you prefer to make your code more readable, although we recommend the lower-case, multiple-lines format. In the examples below the query is smaller just for show up purpose, but it works with any query you need.

Example:
```
<ajax-sparql 
id="request"
url="http://opendata.caceres.es/sparql/"
graph="http://opendata.caceres.es/recurso/salud/centros/"
query="select ?name where{ ?URI a schema:MedicalOrganization. ?URI rdfs:label ?name.}">
</ajax-sparql>

<ajax-sparql 
id="request"
url="http://opendata.caceres.es/sparql/"
graph="http://opendata.caceres.es/recurso/salud/centros/"
query="SELECT ?name WHERE{ ?URI a schema:MedicalOrganization. ?URI rdfs:label ?name.}">
</ajax-sparql>
```

### Retrieval of results.
As we are making an asynchronous request, the data will not be inmediately available. Thanks to the `id` we give to each element, it is easy to access each one in a separate way. To retrieve the data you can use some kind of observer or any other method you prefer, but here we present two options:

* __If our element is included inside another.__  Launch a function in the ready that makes asynchronous calls until some result is received.

Example:
```
<dom-module id="external-component">
  <template>
      <ajax-sparql 
      id="request"
      url="http://opendata.caceres.es/sparql/"
      graph="http://opendata.caceres.es/recurso/salud/centros/"
      query="select ?name where{ ?URI a schema:MedicalOrganization. ?URI rdfs:label ?name.}">
      </ajax-sparql>
  </template>
</dom-module>  

<script>
  Polymer({
    is: 'external-component',
    
    ready: function(){
      this.async(this.checkResults, 300);
    },

    checkResults: function(){
      var result = this.$.request.getResults();
      if (result == undefined){
        this.async(this.checkResults, 300);
      } else {
        //Do what you need with the results:
        console.log(result);
      }
    }
  });
</script>
```

* __If the `ajax-sparql` element is outside any component.__ Create an interval that periodically checks if the there is some response. In affirmative case, stop the interval.

Example:
```
<ajax-sparql 
id="request"
url="http://opendata.caceres.es/sparql/"
graph="http://opendata.caceres.es/recurso/salud/centros/"
query="select ?name where{ ?URI a schema:MedicalOrganization. ?URI rdfs:label ?name.}">
</ajax-sparql>

<script>
  var interval = setInterval(checkResult, 300);

  function checkResult(){
      var result = document.getElementById("request").getResults();
      if (result != undefined){
        clearInterval(interval);
       
        //Do what you need with the results:
        console.log(result);
      }
  }
</script>
```