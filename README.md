## GeoTools

# 1. Visão Geral

Neste artigo, veremos os fundamentos da biblioteca Java de código aberto GeoTools - para trabalhar com dados geoespaciais. Esta biblioteca fornece métodos compatíveis para a implementação de Sistemas de Informação Geográfica (GIS) e implementa e suporta muitos padrões do Open Geospatial Consortium (OGC).

Conforme o OGC desenvolve novos padrões, eles são implementados pelo GeoTools, o que o torna bastante útil para o trabalho geoespacial.

2. Dependências
Precisaremos adicionar as dependências do GeoTools ao nosso arquivo pom.xml. Como essas dependências não estão hospedadas no Maven Central, também precisamos declarar seus repositórios para que o Maven possa baixá-los:

```
<repositories>
    <repository>
        <id>osgeo</id>
        <name>Open Source Geospatial Foundation Repository</name>
        <url>http://download.osgeo.org/webdav/geotools/</url>
    </repository>
    <repository>
        <id>opengeo</id>
        <name>OpenGeo Maven Repository</name>
        <url>http://repo.opengeo.org</url>
    </repository>
</repositories>
```

Depois disso, podemos adicionar nossas dependências:

```
<dependency>
    <groupId>org.geotools</groupId>
    <artifactId>gt-shapefile</artifactId>
    <version>15.2</version>
</dependency>
<dependency>
    <groupId>org.geotools</groupId>
    <artifactId>gt-epsg-hsql</artifactId>
    <version>15.2</version>
</dependency>
```

# 3. GIS e Shapefiles
Para ter algum uso prático da biblioteca GeoTools, precisaremos saber algumas coisas sobre sistemas de informação geográfica e shapefiles.

### 3.1. GIS
Se quisermos trabalhar com dados geográficos, precisaremos de um sistema de informações geográficas (SIG). Este sistema pode ser usado para apresentar, capturar, armazenar, manipular, analisar ou gerenciar dados geográficos.

Parte dos dados geográficos é espacial - faz referência a localizações concretas na terra. Os dados espaciais geralmente são acompanhados pelos dados de atributo. Os dados de atributos podem ser quaisquer informações adicionais sobre cada um dos recursos espaciais.

Um exemplo de dados geográficos seriam as cidades. A localização real das cidades são os dados espaciais. Dados adicionais, como o nome da cidade e a população, formariam os dados do atributo.

### 3.2. Shapefiles
Diferentes formatos estão disponíveis para trabalhar com dados geoespaciais. Raster e vetor são os dois principais tipos de dados.

Neste artigo, veremos como trabalhar com o tipo de dados vetoriais. Este tipo de dados pode ser representado como pontos, linhas ou polígonos.

Para armazenar dados vetoriais em um arquivo, usaremos um shapefile. Este formato de arquivo é usado ao trabalhar com o tipo de dados vetoriais geoespaciais. Além disso, é compatível com uma ampla gama de software GIS.

Podemos usar GeoTools para adicionar recursos como cidades, escolas e pontos de referência aos shapefiles.

# 4. Criação de recursos
A documentação do GeoTools especifica que um recurso é qualquer coisa que pode ser desenhada em um mapa, como uma cidade ou algum ponto de referência. E, como mencionamos, uma vez criados, os recursos podem ser salvos em arquivos chamados shapefiles.

### 4.1. Manter dados geoespaciais
Antes de criar um elemento, precisamos saber seus dados geoespaciais ou as coordenadas de longitude e latitude de sua localização na Terra. Quanto aos dados de atributo, precisamos saber o nome do recurso que queremos criar.

Essas informações podem ser encontradas na web. Alguns sites como simplemaps.com ou maxmind.com oferecem bancos de dados gratuitos com dados geoespaciais.

Quando sabemos a longitude e latitude de uma cidade, podemos facilmente armazená-los em algum objeto. Podemos usar um objeto Map que conterá o nome da cidade e uma lista de suas coordenadas.

Vamos criar um método auxiliar para facilitar o armazenamento de dados dentro do nosso objeto Map:

```
private static void addToLocationMap(
  String name,
  double lat,
  double lng,
  Map<String, List<Double>> locations) {
    List<Double> coordinates = new ArrayList<>();

    coordinates.add(lat);
    coordinates.add(lng);
    locations.put(name, coordinates);
}
```

Agora vamos preencher nosso objeto Map:

```
Map<String, List<Double>> locations = new HashMap<>();

addToLocationMap("Bangkok", 13.752222, 100.493889, locations);
addToLocationMap("New York", 53.083333, -0.15, locations);
addToLocationMap("Cape Town", -33.925278, 18.423889, locations);
addToLocationMap("Sydney", -33.859972, 151.211111, locations);
addToLocationMap("Ottawa", 45.420833, -75.69, locations);
addToLocationMap("Cairo", 30.07708, 31.285909, locations);
```

Se baixarmos algum banco de dados CSV que contenha esses dados, podemos criar facilmente um leitor para recuperar os dados em vez de mantê-los em um objeto como este.

### 4.2. Definindo Tipos de Característica
Então, agora temos um mapa das cidades. Para poder criar recursos com esses dados, primeiro precisamos definir seu tipo. GeoTools oferece duas maneiras de definir tipos de recursos.

Uma maneira é usar o método createType da classe DataUtilites:

```
SimpleFeatureType TYPE = DataUtilities.createType(
  "Location", "location:Point:srid=4326," + "name:String");
```

Outra maneira é usar um SimpleFeatureTypeBuilder, que fornece mais flexibilidade. Por exemplo, podemos definir o Sistema de Referência de Coordenadas para o tipo e podemos definir um comprimento máximo para o campo de nome:

```
SimpleFeatureTypeBuilder builder = new SimpleFeatureTypeBuilder();
builder.setName("Location");
builder.setCRS(DefaultGeographicCRS.WGS84);

builder
  .add("Location", Point.class);
  .length(15)
  .add("Name", String.class);

SimpleFeatureType CITY = builder.buildFeatureType();
```

Ambos os tipos armazenam as mesmas informações. A localização da cidade é armazenada como um ponto e o nome da cidade é armazenado como uma string.

Você provavelmente notou que as variáveis de tipo TYPE e CITY são nomeadas com letras maiúsculas, como constantes. Variáveis de tipo devem ser tratadas como variáveis finais e não devem ser alteradas após serem criadas, então esta forma de nomenclatura pode ser usada para indicar exatamente isso.

### 4.3. Criação de recursos e coleções de recursos
Assim que tivermos o tipo de recurso definido e tivermos um objeto que possui os dados necessários para criar recursos, podemos começar a criá-los com seu construtor.

Vamos instanciar um SimpleFeatureBuilder fornecendo nosso tipo de recurso:

```
SimpleFeatureBuilder featureBuilder = new SimpleFeatureBuilder(CITY);
```

Também precisaremos de uma coleção para armazenar todos os objetos de recurso criados:

```
DefaultFeatureCollection collection = new DefaultFeatureCollection();
```

Como declaramos em nosso tipo de recurso manter um ponto para a localização, precisaremos criar pontos para nossas cidades com base em suas coordenadas. Podemos fazer isso com o JTSGeometryFactoryFinder do GeoTools:

```
GeometryFactory geometryFactory
  = JTSFactoryFinder.getGeometryFactory(null);
```

Observe que também podemos usar outras classes de geometria, como linha e polígono.

Podemos criar uma função que nos ajudará a colocar recursos na coleção:

```
private static Function<Map.Entry<String, List<Double>>, SimpleFeature>
  toFeature(SimpleFeatureType CITY, GeometryFactory geometryFactory) {
    return location -> {
        Point point = geometryFactory.createPoint(
           new Coordinate(location.getValue()
             .get(0), location.getValue().get(1)));

        SimpleFeatureBuilder featureBuilder
          = new SimpleFeatureBuilder(CITY);
        featureBuilder.add(point);
        featureBuilder.add(location.getKey());
        return featureBuilder.buildFeature(null);
    };
}
```

Assim que tivermos o construtor e a coleção, usando a função criada anteriormente, podemos criar recursos e armazená-los em nossa coleção:

```
locations.entrySet().stream()
  .map(toFeature(CITY, geometryFactory))
  .forEach(collection::add);
```

A coleção agora contém todos os recursos criados com base em nosso objeto Mapa que continha os dados geoespaciais.

# 5. Criação de um DataStore
GeoTools contém uma API DataStore que é usada para representar uma fonte de dados geoespaciais. Essa fonte pode ser um arquivo, um banco de dados ou algum serviço que retorna dados. Podemos usar um DataStoreFactory para criar nosso DataStore, que conterá nossos recursos.

Vamos definir o arquivo que conterá os recursos:

```
File shapeFile = new File(
  new File(".").getAbsolutePath() + "shapefile.shp");
```

Agora, vamos definir os parâmetros que vamos usar para dizer ao DataStoreFactory qual arquivo usar e indicar que precisamos armazenar um índice espacial quando criamos nosso DataStore:

```
Map<String, Serializable> params = new HashMap<>();
params.put("url", shapeFile.toURI().toURL());
params.put("create spatial index", Boolean.TRUE);
```

Vamos criar o DataStoreFactory usando os parâmetros que acabamos de criar e usar essa fábrica para criar o DataStore:

```
ShapefileDataStoreFactory dataStoreFactory
  = new ShapefileDataStoreFactory();

ShapefileDataStore dataStore 
  = (ShapefileDataStore) dataStoreFactory.createNewDataStore(params);
dataStore.createSchema(CITY);
```

# 6. Escrevendo em um Shapefile

A última etapa que precisamos fazer é gravar nossos dados em um arquivo de forma. Para fazer isso com segurança, vamos usar a interface de transação que faz parte da API GeoTools.

Essa interface nos dá a possibilidade de enviar facilmente nossas alterações para o arquivo. Ele também fornece uma maneira de realizar uma reversão das alterações malsucedidas se ocorrer algum problema ao gravar no arquivo:

```
Transaction transaction = new DefaultTransaction("create");

String typeName = dataStore.getTypeNames()[0];
SimpleFeatureSource featureSource
  = dataStore.getFeatureSource(typeName);

if (featureSource instanceof SimpleFeatureStore) {
    SimpleFeatureStore featureStore
      = (SimpleFeatureStore) featureSource;

    featureStore.setTransaction(transaction);
    try {
        featureStore.addFeatures(collection);
        transaction.commit();

    } catch (Exception problem) {
        transaction.rollback();
    } finally {
        transaction.close();
    }
}
```

O SimpleFeatureSource é usado para ler recursos e o SimpleFeatureStore é usado para acesso de leitura / gravação. Está especificado na documentação do GeoTools que usar o método instanceof para verificar se podemos gravar no arquivo é a maneira correta de fazê-lo.

Este shapefile pode ser aberto posteriormente com qualquer visualizador GIS que tenha suporte para shapefile.

# 7. Conclusão
Neste artigo, vimos como podemos usar a biblioteca GeoTools para fazer alguns trabalhos geoespaciais muito interessantes.

Embora o exemplo seja simples, ele pode ser estendido e usado para criar shapefiles ricos para vários fins.

Devemos ter em mente que GeoTools é uma biblioteca vibrante e este artigo serve apenas como uma introdução básica à biblioteca. Além disso, o GeoTools não se limita a criar apenas tipos de dados vetoriais - ele também pode ser usado para criar ou trabalhar com tipos de dados raster.
