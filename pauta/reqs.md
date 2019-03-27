Plan de capacitación

se necesita una aplicación en un lenguage como Java que cumpla con los siguientes requisitos:

* Simple. Algo un poco mas complejo que un helloworld
* Que tenga una interfaz web con un logo de tipo Hello World
* La aplicación deb estar contenerizada. funcionalidad por definir.

* Es decir debe existir un proceso de compilación y contenerización. el objetivo de esto es usar el proceso el servicio codebuild
* La aplicación debe pasar por analisis estático de código (SonarQube). 
	* El código debe incluir algunas malas prácticas que hagan que el proceso falle por no pasar el Quality Gate
	* El código puede ser un componente de tipo microserviocio como Springboot o algo adhoc
* El código debe ejecutar una proceso de JUnits con posibilidad con plan de fallo
* El despliegue debe hacerse en modalidad Blue/Blue green u+
* sando ECS + Fargate.

**Documentacion en formato Markdown**
