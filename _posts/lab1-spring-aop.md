layout: post
title: "POST TITLE"
date: YYYY-MM-DD hh:mm:ss -0000
categories: CATEGORY-1 CATEGORY-2
--- 

## Spring Boot AOP

En esta ocasion aplicaremos el paradigma de Programación Orientada a Aspectos o AOP por sus siglas en inglés Aspect Oriented Programming.

Este paradigma nos permite separar las funcionalidades del sistema, en donde las funcionalidades generales del sistema se pueden usar traansversalmente y asi dejar las funciones modulares en cada uno de los modulos del sistema.

Actualmente Spring Boot soporta este paradigma a traves de la dependencia 
```
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

Puede obtener mayor información de AOP para Spring Boot puede consultar el siguiente enlace  [Aspect Oriented Programming with Spring](https://docs.spring.io/spring-framework/docs/4.3.15.RELEASE/spring-framework-reference/html/aop.html)


Vamos a realizar un pequeño ejemplo de como emplear AOP para capturar la ejecución de un metodo y capturar el punto de cruce `@Around` y agregar un registro en el log con el tiempo que demoro la ejecución del metodo.


# Entorno
- maven.
-  java 11
- IntetelliJ CE


## Creando un `@Aspect`

Para crear un aspect debemos colocar la annotación @Aspect en la declaración de la clase.

```
@Aspect
@Service
public class LoggingAspect {
```

Con esta anotación le estamos indicando a Spring que esta clase aplicará ejecuciones de aspectos.


## Creando un `@Around`

Para indicarle a Spring que queremos capturar el punto de cruce antes de la ejecución del metodo debemos realizar las siguientes acciones.


> Primero creamos el metodo que deseamos aplicar en nuestro aspecto.

```
public Object logging(ProceedingJoinPoint joinPoint){
        LOGGER.info("Executing Around method: {}", joinPoint.getSignature().getName());
        long startTime = System.currentTimeMillis();
        Object retVal = joinPoint.proceed();
        long endTime = System.currentTimeMillis();
        long time = endTime - startTime;
        LOGGER.info("Took {} seg.", (time/1000));
        return retVal;
}
```


> Ahora agregamos nuestra anotación `@Around` al metodo creado.


```
@Around
public Object logging(ProceedingJoinPoint joinPoint){
        LOGGER.info("Executing Around method: {}", joinPoint.getSignature().getName());
        long startTime = System.currentTimeMillis();
        Object retVal = joinPoint.proceed();
        long endTime = System.currentTimeMillis();
        long time = endTime - startTime;
        LOGGER.info("Took {} seg.", (time/1000));
        return retVal;
}
```

> Por ultimo timo indicamos el punto de cruce en el cual deseamos que se ejecute nuestro metodo. En este caso usaremos `execution(* com.hvs..*(..))` para indicar que se dispare para cualquier ejecución de un metodo dentro de nuestro paquete principal `com.hvs` con cualquier argumento.

```
@Around(execution("* com.hvs..*(..))")
public Object logging(ProceedingJoinPoint joinPoint){
        LOGGER.info("Executing Around method: {}", joinPoint.getSignature().getName());
        long startTime = System.currentTimeMillis();
        Object retVal = joinPoint.proceed();
        long endTime = System.currentTimeMillis();
        long time = endTime - startTime;
        LOGGER.info("Took {} seg.", (time/1000));
        return retVal;
}
```

## Ejecutando la aplicación

Para validar este aspecto hemos creado un controlador de spring boot el cual calcula un valor random dentro de una rnago de dos numeros que recibe como parametros.

```
@GetMapping(value = {"/${action-delay}/{min}/{max}"})
public long delay(@PathVariable long min, @PathVariable long max) {

        LOGGER.info("into my method");
        if (min == 0) {
            min = defaultMin;
        }
        if (max == 0) {
            max = defaultMax;
        }
        if (min < 0){
            throw new IllegalArgumentException("min value should not be < 0");
        }
        if (max < min) {
            throw new IllegalArgumentException("max value should not be bigger that min value");
        }
        long delay = min + (long) (Math.random() * (max - min));
        sleep((delay * 1000));
        return delay;
 }
```

Luego, solo debemos ejecutar nuestra aplicación, hacer el consumo del controllador y comprobar que en nuestro log se visualice el tiempo que tomo la ejecución del metodo.

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.5.4)

2021-10-07 14:16:10.027  INFO 4060142 --- [           main] com.hvs.lab.aop.AOPApplication           : Starting AOPApplication using Java 11.0.11 on cotetec099 with PID 4060142 (/home/hmartinez/IdeaProjects/lab-1-aop-spring/target/classes started by hmartinez in /home/hmartinez/IdeaProjects/lab-1-aop-spring)
2021-10-07 14:16:10.028  INFO 4060142 --- [           main] com.hvs.lab.aop.AOPApplication           : No active profile set, falling back to default profiles: default
2021-10-07 14:16:11.499  INFO 4060142 --- [           main] o.s.b.web.embedded.netty.NettyWebServer  : Netty started on port 8080
2021-10-07 14:16:11.505  INFO 4060142 --- [           main] com.hvs.lab.aop.AOPApplication           : Started AOPApplication in 2.11 seconds (JVM running for 2.701)
2021-10-07 14:16:14.533  INFO 4060142 --- [or-http-epoll-3] com.hvs.lab.aop.aspect.LoggingAspect     : Executing Around method: delay
2021-10-07 14:16:14.542  INFO 4060142 --- [or-http-epoll-3] c.h.l.a.c.i.DelayRandomController        : into my method
2021-10-07 14:16:19.542  INFO 4060142 --- [or-http-epoll-3] com.hvs.lab.aop.aspect.LoggingAspect     : Took 5 seg.

```

Al final de la ejecución podemos ver `Took 5 seg.` indicandonos que tomo 5 segundo la ejecución del metodo con lo cual comprobamos que el aspecto se ejecuto correctamente.

Puede ver el código completo de este ejemplo [aquí](https://github.com/martinezhenry/lab-1-aop-spring)
