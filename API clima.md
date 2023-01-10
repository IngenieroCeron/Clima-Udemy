#  Pasos para crear Api de clima

Primero hice la app a partir del curso de UDEMY pero ahi lo hacen con Storyboar, entonces yo lo hice apartir de SwuiftUI

Para ello primero cree la interfaz parecida a la que utilizan

Cree las variables con State para la imagen, los grados y para la localidad

Tambien cree las insancias para poder usar modo dark o light:
    Para cambiar la apariencia con este boton primero tenemos que declarar una propiedad en el archivo principal con un -AppStorage igual al creado en el archivo ContentView.
        -Luego en ese mismo archivo(en el principal), le agregamos un modificador al ContentView() .preferredColorScheme
        -Despues condicionamos el background del textfield y por ultimo condicionamos cualquier cosa que queramos.
        
Despues en el video #148 Angela explica lo que es una API:
    Api: Conjunto de comndos, funciones, protocolos y objetos. Y las API se pueden usar para crear software o para interactúar con un sistema externo.
    Proporciona a los desarrolladores comandos estándar para realizar operaciones comunes para que no tengan que escribir el código desde cero.
    Así que una API se puede considerar como simplemente un contrato entre el desarrollador y el proveedor de API.
    Entonces si queremos llenar nuestra aplicación con datos de clima, podemos usar los mapas de OpenWather y así llenar nuestra app con la info de esa API.
    
Despues conocimos la página OpenWeather (https://openweathermap.org/api) de ahi vamos a sacar toda la info. Tuvimos que regsitrarnos gratis. Una vez que te registras, en la pestaña "API Keys" ahi aparece mi clave unica (2dd64d7bac933ea72e62582861bdc014) con el que voy a poder comunicarme. Para saber como usar el servicio API nos vamos a la pagina antes mencionada y ahi en la pestañita "API doc" le damos clic para ver la documentacion, ahi observamos que podemos obtener el clima por ciudad, por id, por coordenadas o por código postal. En mi caso no aparecio asi la pagina, aparecio diferente, yo creo por que ya esta actualizada.

Ahi en esa pagina viene como hacer el API call, te proporcionan un link, donde debes poner la ciudad y tu numero de clave unica:
        https://api.openweathermap.org/data/2.5/weather?q={city name}&appid={API key}
        
        Entonces ahora ponemos nuestra clave despues del appid=:
        https://api.openweathermap.org/data/2.5/weather?q=london&appid=2dd64d7bac933ea72e62582861bdc014
        
        ingresando esa liga en el navegador nos va a direccionar a una página donde sale el json pero con una estructura medio chusca:
        {"coord":{"lon":-0.1257,"lat":51.5085},"weather":[{"id":803,"main":"Clouds","description":"broken clouds","icon":"04n"}],"base":"stations","main":{"temp":279.89,"feels_like":278.06,"temp_min":276.83,"temp_max":282.01,"pressure":1037,"humidity":82},"visibility":9000,"wind":{"speed":2.57,"deg":220},"clouds":{"all":62},"dt":1647557380,"sys":{"type":2,"id":2019646,"country":"GB","sunrise":1647497428,"sunset":1647540444},"timezone":0,"id":2643743,"name":"London","cod":200}
        
        Entonces para poder ver una estructura de arbol y sea mas entendible; necesitamos agregar una extension de json viewer en la liga:
        https://chrome.google.com/webstore/detail/json-viewer-pro/eifflpmocdbdmepbjaopkkhbfmdgijcc
        
        Una vez que lo instale, al hacer refresh la pagina ya lo detecta en automatico y ya se ve bonis.
        
        Para que salga en grados celsius le agregamos un parametro units=metric:
        https://api.openweathermap.org/data/2.5/weather?q=london&appid=2dd64d7bac933ea72e62582861bdc014&units=metric
        
        Ahora para hacer el call desde la app, vamos a crear un nuevo archivo swift en la carpeta model y lo llamamos WeatherManager.
        
        Dentro del archivo que acabamos de crear agregamos una estructura y agregamos la primer constante string por el momento con el link anterior. Ahora la parte de este link que va a cambiar va a ser la parte de la locacion, ahorita tenemos london. Entonces lo que vamos a hacer es eliminar la parte del parametro de london (q=london&), y despues con codigo le agregamos ese parametro dependiendo de la localidad que escoja el usuario. La liga queda así:
                struct WeatherManager {
                    let weatherURL = "https://api.openweathermap.org/data/2.5/weather?appid=2dd64d7bac933ea72e62582861bdc014&units=metric"
                }

        Despues vamos a crear un metodo y lo nombramos fetchWeather el cual va a recibir una entrada del nombre de la ciudad que elija el usuario:
            func fetchWeather (cityName: String){
                let urlString = "\(weatherURL)&q=\(cityName)"
            }
            
        Luego agregamos una instancia de la estructura en el contentview:
                var weatherManager = WeatherManager()
                
        Despues en el boton de buscar (la lupita) vamos a poner una variable llamada city igualada a serachLocation y despues creamos un condicionante para que cuando esté vacio el textfield mande una alerta de campo vacio o de lo contrario que mande a llamar a la funcion antes creada y se agregue la ciudad a la variable url:
                        let city = searchLocation
                        if city.isEmpty {
                            isCityEmpty = true
                        } else {
                            weatherManager.fetchWeather(cityName: city)
                        }
                        ** se me olvido mencionar que hay que agregar l variable booleana para la alerta:
                                @State var isCityEmpty: Bool = false
                        
                Y para la alerta le agregamos al final del Button el codigo para la alerta:
                        .alert(isPresented: $isCityEmpty) {
                            Alert(title: Text("Error"), message: Text("No se encontró la localidad"), dismissButton: .default(Text("Entendido")))
                        }
        Ahora ya tenemos el link completo para hacer la busqueda. Y todo lo que queda por hacer es ir a traves de internet desde nuestra app y recuperar los datos reales
        
Entonces para hacer todo el networking que nos falta, es decir mandar el request al webservice y que nos regrese la info tenemos una serie de pasos:
        ## 1. Crear un objeto URL
        ## 2. Crear una sesión URL
        ## 3. Asignar una tarea a la URLSession para obtener(fetch) los datos de esa fuente en particular
        ## 4. Por ultimo crear iniciar la tarea (cuando apretamos enter en el boton de busqueda que activa todo el proceso de red (networking))
            
    Para hacer lo anterior:
        1. crear otro metodo en la estructura WeatherManager y lo llamamos performRequest
        2. El metodo recibirá la variable urlString que creamos en el metodo fetchWeather, por lo que en este ultimo metodo hay que pasarle el parametro así: performRequest(urlString: urlString)
        3. Dentro de este nuevo metodo (performRequest) haremos los 4 pases antes mencionados, quedando así el codigo:
        func performRequest(urlString: String){
        
        // 1. Crear un objeto URL
        
        if let url = URL(string: urlString) {
            
            // 2. Crear una sesión URL
            
            let session = URLSession(configuration: .default)
            
            // 3. Asignar una tarea a la URLSession para obtener(fetch) los datos de esa fuente en particular
            // Para este punto en la parte completionHandlre espera una entrada de tipo funcion, por que vamos a crear una nueva funcion afuera de esta funcion llamada handle() que recibe 3 parametros: data, response y error. Entonces desde el completionHandler tenemos que mandarle esas 3 parametros. Pero esos 3 parametros nosotros no los definimos, esos 3 parametros se crean a partir de que se activa la tarea, entonces los dejamos en blanco como si no tuvieran ningun valor, van a tener valor pero cuando la variable task se active.
            
            let task = session.dataTask(with: url, completionHandler: handle(data: response: error:))
            
            // 4. Por ultimo crear iniciar la tarea (cuando apretamos enter en el boton de busqueda que activa todo el proceso de red (networking))
            
            task.resume()
            
        }
    }
    // esta funcion es la que manejará los datos que la variable tarea le pase una vez que se inicie la variable task.
    func handle(data: Data?, response: URLResponse?, error: Error?) {
        if error != nil {
            print(error!) // aqui unwrappe el error por que con el if de arriba ya estamos comprobando que si tenga información
            return // al agregarle esta palabra simplemente la funcion se detiene y no sigue leyendo las siguientes lineas de codigo
        }
        
        // con este if estamos guardando los datos del request y los convertimos en tipo String.
        if let safeData = data {
            let dataString = String(data: safeData, encoding: .utf8)
            print(dataString)
        }
        
    }

Despues de hacer este codigo ya tenemos los datos guardados en la variable dataString



