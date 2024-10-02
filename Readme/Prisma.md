[[SQL]]

[Curso introductorio](https://www.youtube.com/watch?v=RebA5J-rlwg)

Prisma **es un kit de herramientas (toolkit) de base de datos de código abierto**. Reemplaza los ORM tradicionales y facilita el acceso a la base de datos con un generador de consultas o query builder automáticamente generado y type-safe que se adapta a nuestro esquema de base de datos.

Un ORM (**Object Relational Mapping**) es un modelo de programación cuya misión es transformar las tablas de una base de datos de forma que las tareas básicas, que realizan los programadores, estén simplificadas. Al hacer esto, se puede ahorrar mucho tiempo, ya que cada vez la información es mayor y más cambiante.

[[Prisma y los ORMs|Más sobre Prisma y los ORMs]]

[[Modificadores de tipo del modelo]]

1. Crear carpeta "prisma" y dentro abrir VS Code.
2. Abrir la consola y tipear `npm init -y`para inicializar el package.json del proyecto aceptando los valores por defecto con la -y. 
3. `npm i --save-dev prisma typescript ts-node @types/node nodemon` porque toda dependencia de este proyecto es dev. "typescript" porque prisma está construido sobre el mismo aunque no necesitas usarlo. "ts-node" da definiciones de ts para node. @types/node ayuda con los tipos en node. nodemon para actualizar los cambios en tiempo real.
4. Crear tsconfig.json en el proyecto, que incluirá toda la config de ts. 
	```ts
	{
	  "compilerOptions": {
		"sourceMap": true,
		"outDir": "dist",
		"strict": true,
		"lib": ["esnext"],
		"esModuleInterop": true
	  }
	}
	```
   Se puede encontrar en la documentación de prisma esta config pero se puede cambiar según tus necesidades.
5. `npx prisma init --datasource-provider postgresql` para inicializar prisma. También en este caso le pasamos qué BBDD usaremos.
6. Instalamos la extensión oficial de prisma (Ctrl + Shift + X)
7. Agregamos esta config a settings.json: 
	```json
	"[prisma]": {
	"editor.defaultFormatter": "Prisma.prisma",
	"editor.formatOnSave": true
	}
   ```
8. Como alternativa está el comando por terminal `npx prisma format`, pero este hay que ejecutarlo manualmente cada vez que queremos darle el formato.
9. En el archivo .env que se generó, en la url, creamos nuestras credenciales. En este caso lo dejé así: DATABASE_URL="postgresql://johndoe:password@localhost:5433/test"
   La última palabra tiene que ser una BBDD que hayamos creado.
10. Al final de nuestro schema.prisma, definimos nuestra primer modelo (que prisma traduce a tabla para interactuar con la BBDD):
    ```prisma
    model User {
    id   Int    @id @default(autoincrement())
    name String
	}
	```
	Donde @id es nuestra clave primaria y si no recibe valor, por defecto es auto incremental.
	[[Modelos en Prisma|Modelos en Prisma]]
11. Para que prisma aplique los cambios de nuestro schema a la BBDD debemos correr el comando `npx prisma migrate dev --name init`.
    Esto creará la carpeta "migrations" con la subcarpeta de esta migracion "fecha_name" y dentro el archivo en concreto en formato sql con el nombre "migration.sql".
12. Si aún no estaba creada la BBDD, dará error, por lo cual acá dejo los pasos para [[Instalar Postgresql]]
13. Yo con el usuario creado no logré crear la migración de prisma pero luego configuré el super user "postgres" y con ese pude:
    1. `sudo -u postgres psql` en la terminal de Linux para correr postgresql
    2. `ALTER USER postgres PASSWORD 'nueva_contraseña';` para establecerle la password al super user
    3. En el .env de prisma actualizo la... `DATABASE_URL="postgresql://postgres:tu_contraseña@localhost:5432/test"`
    4. Vuelvo a migrar con `npx prisma migrate dev --name init`.
14. `npm i @prisma/client` en VS Code terminal para instalar el cliente de prisma.
15. `npx prisma generate`cada vez que necesite generar nuevamente el cliente con los cambios que haya hecho en el `schema.prisma`.
16. En la raíz del proyecto creamos la carpeta `script.ts` y pegamos:
    ```sql
    import { PrismaClient } from '@prisma/client'
	const prisma = new PrismaClient()
	```
	Esto nos permite ingresar a nuestro cliente prisma.
17. En el mismo archivo agregamos debajo:
    ```ts
    async function main() {
  // ... you will write your Prisma Client queries here
	}

	main()
	  .catch((e) => {
	    console.error(e.message);
	  })
	  .finally(async () => {
	    await prisma.$disconnect();
	  });
	```
18. Agregamos a `package.json` el script para ejecutar con nodemon:
 ```json
 "scripts": {
    "devStart": "nodemon script.ts"
  }, 
  ```
  Esto va a ejecutar el `script.ts` automáticamente por cada cambio que hagamos (al guardar el archivo).
19. [Recapitulando](https://youtu.be/RebA5J-rlwg?si=upwXB-uAGTq59H96&t=786)
20. Terminamos de completar las entidades User y Posts:
	```sql (prisma)
	    generator client {
	    provider = "prisma-client-js"
	}
	datasource db {
		provider = "postgresql"
		url      = env("DATABASE_URL")
	}
	
	model User {
		id          Int     @id @default(autoincrement())
		name        String
		email       String
		isAdmin     Boolean
		preferences Json
		posts       Post[]
	}
	
	model Post {
		rating    Float
		createdAt DateTime
		updatedAt DateTime
		author    User     @relation(fields: [authorId], references: [id])
		authorId  Int
	}
		```
	Donde @relation es nuestra clave foránea, osea la que relaciona las entidades y a la cual le pasamos como parámetros los campos que relacionan esas entidades.
21. Volviendo a la clave primaria, la forma segura de hacerlo es usando `uuid`:
    ```sql
    model User {
    id          String  @id @default(uuid())
    name        String
    email       String
    isAdmin     Boolean
    preferences Json
    posts       Post[]
	}
	
	model Post {
		id        String  @id @default(uuid())
	    rating    Float
	    createdAt DateTime
	    updatedAt DateTime
	    author    User     @relation(fields: [authorId], references: [id])
	    authorId  String
	}
	```
	Reemplazamos `autoincrement()`por `uuid()`y el tipo de dato de id a `String`. También en `authorId` -> `String`.
	También le agregamos un id a `Post`, que requería también clave primaria.
22. Modificamos a versión de posts favoritos y posts escritos:
    ```sql (prisma)
	model User {
		id             String  @id @default(uuid())
		name           String
		email          String
		isAdmin        Boolean
		preferences    Json
		writtenPosts   Post[]  @relation("WrittenPosts")
		favoritedPosts Post[]  @relation("FavoritedPosts")
	}
	
	model Post {
		id            String   @id @default(uuid())
		rating        Float
		createdAt     DateTime
		updatedAt     DateTime
		author        User     @relation("WrittenPosts", fields: [authorId], references: [id])
		authorId      String
		favoritedVBy  User?     @relation("FavoritedPosts", fields: [favoritedById], references: [id])
		favoritedById String?
	}
	```
23. Agregamos el modelo`Category` (relación muchos a muchos):
  ```sql
	model Post {
	    id            String     @id @default(uuid())
	    rating        Float
	    createdAt     DateTime
	    updatedAt     DateTime
	    author        User       @relation("WrittenPosts", fields: [authorId], references: [id])
	    authorId      String
	    favoritedVBy  User?      @relation("FavoritedPosts", fields: [favoritedById], references: [id])
	    favoritedById String?
	    categories    Category[]
	}
	
	model Category {
	    id    String @id @default(uuid())
	    posts Post[]
	}
	```
	En este caso prisma reconoce la relación de forma automáitca y por detrás de escena establece la JOIN TABLE de sql.
24. Establecemos modelo para preferencias del usuario (relación uno a uno):
    ```sql
    model User {
	    id             String          @id @default(uuid())
	    name           String
		email          String          @unique
	    isAdmin        Boolean
	    preferences    Json
	    writtenPosts   Post[]          @relation("WrittenPosts")
	    favoritedPosts Post[]          @relation("FavoritedPosts")
	    UserPreference UserPreference?
	}

	model UserPreference {
	    id           String  @id @default(uuid())
	    emailUpdates Boolean
	    user         User    @relation(fields: [userId], references: [id])
	    userId       String @unique
	}
	```
	En este caso el atributo del userId de las preferencias debe ser único.
25. Para la actualización automática de los campos "creado" y "actualizado", veremos algunos de los atributos restantes...
    `@updatedAt y @default(now())`
    ```sql
    createdAt     DateTime   @default(now())
    updatedAt     DateTime   @updatedAt
    ```
26. Atributos de bloque:
    Son los que definen reglas para todo el bloque, como agregaremos en el siguiente ejemplo...
    ```sql   
	model User {
	    id             String          @id @default(uuid())
	    age            Int
	    name           String
	    email          String          @unique
	    isAdmin        Boolean
	    preferences    Json
	    writtenPosts   Post[]          @relation("WrittenPosts")
	    favoritedPosts Post[]          @relation("FavoritedPosts")
	    userPreference UserPreference?
	
	    @@unique([age, name])
	    @@index([email])
	}

	model Post {
	    // id            String     @id @default(uuid())
	    title         String
	    averageRating Float
	    createdAt     DateTime   @default(now())
	    updatedAt     DateTime   @updatedAt
	    author        User       @relation("WrittenPosts", fields: [authorId], references: [id])
	    authorId      String
	    favoritedVBy  User?      @relation("FavoritedPosts", fields: [favoritedById], references: [id])
	    favoritedById String?
	    categories    Category[]
	
	    @@id([title, authorId])
	}
    ``` 
    En el caso de @@unique, establecemos que la combinación de age y name debe ser única.
    En el caso de @@index, estamos estableciendo un índice para el email para que si queremos luego ordenar por email sea mejor en rendimiento el indexado.
    En el caso de @@id, que luego lo borraremos en este ejercicio, sería para establecer que el id se forme de la combinación del title y el author (comentamos el id anterior).
27. Roles:
    También podemos enumerar roles:
    ```sql
    model User {
	    id             String          @id @default(uuid())
	    age            Int
	    name           String
	    email          String          @unique
	    role           Role            @default(BASIC)
	    writtenPosts   Post[]          @relation("WrittenPosts")
	    favoritedPosts Post[]          @relation("FavoritedPosts")
	    userPreference UserPreference?
	
	    @@unique([age, name])
	    @@index([email])
	}
	enum Role {
		BASIC
		EDITOR
		ADMIN
	}
    ```
28. Migremos ahora la nueva info:
    `npx prisma migrate dev` pero nos sale error porque ya teníamos datos en la BBDD que no son congruentes con los nuevos modelos y config. Así que vamos a `script.ts` y cambiamos:
    ```sql
      const user = await prisma.user.create({ data: { name: 'Kyle' } });
  console.log(user);
  ```
  por:
  `await prisma.user.deleteMany()`
  y ejecutamos `npm run devStart`
  ahora sí, hacemos la migración con `npx prisma migrate dev`
  Nos pedirá confirmación y un nombre. Le ponemos test y damos enter.
  29. [[Estructurar consultas (querys) en un ORM]] 
      Hasta ahora esto no lo vi en el curso pero me surgió esa duda y lo consulté.
  30. Ahora sí podemos volver a usar nuestra función de crear usuario pero notamos que no se autocompletan con la sintaxis de prisma los datos. Por lo cual debemos primero ejecutar `npx prisma generate` para asegurarnos que nuestro generador funciona. Apretamos Ctrl + Space dentro del objeto data y vemos que aún no se auto genera. Por lo cual cerrar `script.ts` y volver a abrirlo podría solucionar el problema. Si persiste el problema, hacemos Ctrl + click en la función que estamos tratando de generar y nos llevará a su definición en TS (en este caso Ctrl + click en create). El abrir ese archivo hace que se actualice el script y si ahora hacemos Ctrl + Space de nuevo, veremos las opciones de auto completado. Como última opción si aún no se soluciona, probar de cerrar VS Code y volver a abrirlo. 
31. Actualizamos el script.ts con los siguientes datos: 
    ```ts
	import { PrismaClient } from '@prisma/client';
	const prisma = new PrismaClient();
	
	async function main() {
	  const user = await prisma.user.create({
	    data: {
	      name: 'Kyle',
	      email: 'kyle@test.com',
	      age: 27,
	    },
	  });
	
	  console.log(user);
	}
	
	main()
	  .catch((e) => {
	    console.error(e.message);
	  })
	  .finally(async () => {
	    await prisma.$disconnect();
	  });
	```
	y ejecutamos `npm run devStart`. Veremos por consola que se muestra nuestro nuevo usuario con sus datos.
32. Probamos de (cambiando algo como un espacio extra) volver a guardar el archivo y veremos que da error, porque dice que el email ya existe (intenta volver a crear el mismo usuario). Esto lo resolvemos agregando `  await prisma.user.deleteMany();` justo después de `async function main(){`. Para asegurarnos que cada vez que se ejecute se borre toda la BBDD. Esto es para aprender en este curso, no es común en un escenario real al usar Prisma.
33. Agregamos a la función main preferencias de usuario y la propiedad include para que estén visibles:
	```ts
	async function main() {
	  await prisma.user.deleteMany();
	  const user = await prisma.user.create({
	    data: {
	      name: 'Kyle',
	      email: 'kyle@test.com',
	      age: 27,
	      userPreference: {
	        create: {
	          emailUpdates: true,
	        },
	      },
	    },
	    include: {
	      userPreference: true,
	    },
	  });
	
	  console.log(user);
	}
	```
34. Realizamos los siguientes cambios para evitar el error que sale con esta nueva config....
    ```sql
    model User {
    id               String          @id @default(uuid())
    age              Int
    name             String
    email            String          @unique
    role             Role            @default(BASIC)
    writtenPosts     Post[]          @relation("WrittenPosts")
    favoritedPosts   Post[]          @relation("FavoritedPosts")
    userPreference   UserPreference? @relation(fields: [userPreferenceId], references: [id])
    userPreferenceId String?         @unique

    @@unique([age, name])
    @@index([email])
	}
	
	model UserPreference {
	    id           String  @id @default(uuid())
	    emailUpdates Boolean
	    user         User?
	}
	```
	(Compararlo con los modelos anteriores)
	Veremos que al ir haciendo cambios si se guarda por cambio de pestaña con auto guardado, completará de forma errónea ciertos campos. Ese es el resultado final.
	Con este cambio lo que hicimos es indicar que el usuario opcionalmente puede tener preferencias de usuario.
35. Ahora volvemos a migrar con `npx prisma migrate dev --name test2` y debería funcionar.
    Lo hacemos para que se eliminen tanto el usuario como las preferencias de usuario.
36. Ahora ejecutamos `npm run devStart` y vemos que se eliminó ese usuario y creó uno nuevo pero al tener ahora el include, userPreference aparece en los resultados (con nodemon por consola)
37. Podemos reemplazar el include por un select que nos da muchoas opciones como obtener el nombre de usuario además de sus preferencias:
    ```ts
    select: {
      name: true,
      userPreference: true,
    },
    ```
    Como también podría ser solicitar solo una de las preferencias:
    ```ts
    select: {
      name: true,
      userPreference: { select: { id: true } },
    },
    ```
    **Atención**: A diferencia del curso, a mí me tiraba este [[Error de clave foránea]] el cual resolví agregando `await prisma.userPreference.deleteMany();` justo antes del deleteMany de user.
    De estas dos maneras solo obtuvimos dos propiedades con select en lugar de todo el objeto con include.
38. Podemos también imprimir por consola lo que pasa en el detrás de escena de Prisma, osea las querys:
    `const prisma = new PrismaClient({ log: ['query'] });`
39.  Creamos varios usuarios de esta manera:
 ```ts
	import { PrismaClient } from '@prisma/client';
	const prisma = new PrismaClient({ log: ['query'] });
	
	async function main() {
	await prisma.user.deleteMany();
	const users = await prisma.user.createMany({
		data: [
			{
				name: 'Kyle',
				email: 'kyle@test.com',
				age: 27,
			},
			{
				name: 'Sally',
				email: 'sally@test.com',
				age: 32,
			},
		],
	});
	
		console.log(users);
	}
	
	main()
		.catch((e) => {
			console.error(e.message);
	})
		.finally(async () => {
			await prisma.$disconnect();
		});
	```
40. Ahora nos vamos a enfocar en la lectura de data. En este caso con el método findUnique, que te permite encontrar cosas basadas en claves únicas en tu BBDD:
    ```ts
	async function main() {
	  const user = await prisma.user.findUnique({
		where: {
		  email: 'kyle@test.com',
		},
	  });

	  console.log(user);
	}
	```
	También le podríamos pasar un select o include como en ejemplos anteriores si quisiéramos.
41. En Prisma cuando definimos restricciones de unicidad múltiple como hicimos con `@@unique([age, name])` en nuestro schema, podemos buscar usando ese criterio:
    ```ts
    async function main() {
  const user = await prisma.user.findUnique({
    where: {
      age_name: {
        age: 27,
        name: 'Kyle',
      },
    },
  });

  console.log(user);
	}
	```
42. Si queremos hacer búsqueda de claves que NO son únicas, podemos usar por ej. `findFirst()` para traer la primera coincidencia:
    ```ts
    async function main() {
	  const user = await prisma.user.findFirst({
	    where: {
	      name: 'Sally',
	  },
  });

    console.log(user);
	}
    ``` 
43. O buscar todos los que cumplan ese criterio con `findMany()`
44. Tenemos la opción distinct para que devuelva los que son diferentes. Por ejemplo si pedimos que distinga el nombre, en el caso de tener 3 Sallys en la BBDD solo devolverá la primera con ese nombre ya que las otras 2 no tienen diferente nombre (es un ejemplo de mierda pero es del curso). Pero si en cambio también le decimos que distinga la edad, ahora sí devolverá a todas ya que la combinación de esas claves no ser repite en ellas (tienen diferente edad):
  ```ts
	async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      name: 'Sally',
	    },
	    distinct: ['name', 'age'],
	  });
	
	  console.log(user);
	}
	```
45. También podemos hacer paginación, y decirle que tome solo 2 resultados pasando en el lugar de distinct la propiedad `take: 2`. Esto devolvería solo 2 Sallys. También podemos hacer `take: 2, skip: 1` para que saltee la primera Sally.
46. Podemos agregar la propiedad `orderBy: {age: "asc"}`junto con esas propiedades para que las devuelva ordenadas.
47. Para encontrar la cantidad de usuarios con un determinado nombre podemos en la cláusula where pasar el name con la propiedad equals y en el console log especificar que queremos el user.length:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      name: { equals: 'Sally' },
	    },
	  });

  console.log(user.length);
	}
   ```
48. Tenemos también la cláusula `not` para buscar los que no cumplan esa condición:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      name: { not: 'Sally' },
	    },
	  });

	  console.log(user);
	}
	```
49. Pasar `in` al name nos dará los que cumplan con ese nombre dentro del array:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      name: { in: ['Sally','Kyle'] },
	    },
	  });

	  console.log(user);
	}
	```
	También podría ser `notIn` para que traiga los que no están en el array.
50. Pasamos a la propiedad age la cláusula `lt` que indica `less than` para traer los users con edad inferior a la pedida: 
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      age: { lt: 20 },
	    },
	  });
	
	  console.log(user);
	}
   ```
	   o `gh` para `greater than`, `lte`= `less than or equal to` o `gte` = `greater than or equal to`.
   51. Contains: 
       ```ts
	   async function main() {
		  const user = await prisma.user.findMany({
		    where: {
		      email: { contains: '@test.com' },
		    },
		  });
		
		  console.log(user);
		}
	```
		En lugar de contains también podemos pasar `startsWith`.
52. Podemos pasarlo dentro de un AND para varias condiciones:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      AND: [
	        { email: { startsWith: 'sally' } },
	        { email: { endsWith: '@test1.com' } },
	      ],
	    },
	  });
	
	  console.log(user);
	}
	```
53. Ejemplo con OR:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      OR: [{ email: { startsWith: 'sally' } }, { age: { gt: 20 } }],
	    },
	  });
	
	  console.log(user);
	}
    ```
54. Ejemplo con NOT:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      NOT: { email: { startsWith: 'sally' } },
	    },
	  });
	
	  console.log(user);
	}
	```
55. WHERE con `userPreference`:
    ```ts
    async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      userPreference: {
	        emailUpdates: true,
	      },
	    },
	  });
	
	  console.log(user);
	}
	```
56. Ejemplo con `every`:
    ```ts
	async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      writtenPosts: {
	        every: {
	          createdAt: new Date(),
	        },
	      },
	    },
	  });
	
	  console.log(user);
	}
	```
	otro ejemplo pero con none:
	```ts
	async function main() {
	  const user = await prisma.user.findMany({
	    where: {
	      writtenPosts: {
	        none: {
	          title: 'Test',
	        },
	      },
	    },
	  });
	
	  console.log(user);
	}
	```
	También podemos poner `some`.
57. `is` & `isNot`:
    ```ts
    async function main() {
	  const user = await prisma.post.findMany({
	    where: {
	      author: {
	        is: {
	          age: 27,
	        },
	        isNot: {
	          name: 'Sally',
	        },
	      },
	    },
	  });
	
	  console.log(user);
	}
	```
58. `update`:
    ```ts
    async function main() {
	  const user = await prisma.user.update({
	    where: {
	      email: 'sally@test.com',
	    },
	    data: {
	      email: 'sally@test3.com',
	    },
	  });
	
	  console.log(user);
	}
	```
	Nota: Los updates tienen que ser en campos únicos como el mail en este  caso.
	Otro ejemplo:
	```ts
	async function main() {
  const user = await prisma.user.update({
    where: {
      email: 'kyle@test.com',
    },
    data: {
      age: {
        decrement: 1,
      },
    },
  });

  console.log(user);
	}
	```
59. `updateMany`:
    ```ts
    async function main() {
	  const user = await prisma.user.updateMany({
	    where: {
	      name: 'Sally',
	    },
	    data: {
	      name: 'New Sally',
	    },
	  });
	
	  console.log(user);
	}
	```
	Nota: Update many no permite pasarle select o include.
60. Conectar diferentes relaciones con update:
    1. Así creamos la nueva preferencia de usuario como ya hemos visto:
	    ```ts
	    async function main() {
		  const user = await prisma.user.update({
		    where: {
		      email: 'kyle@test.com',
		    },
		    data: {
		      userPreference: {
		        create: {
		          emailUpdates: true,
		        },
		      },
		    },
		  });
		
		  console.log(user);
		}
	    ```
	2. Pero también podemos conectar una preferencia de usuario existente a un usuario existente. En teoría por mas que el data marque
	3. error, en el curso pudo crear la preferencia por separado para después conectarla. A mi no me salió el resultado por consola pero este es el código:
	  ```ts
	  async function main() {
		  const preference = await prisma.userPreference.create({
		    data: {
		      emailUpdates: true,
		    },
		  });
		
		  console.log(preference);
		}
	   ```
   3. Realizamos la conexión:
      ```ts
      async function main() {
		  const user = await prisma.user.update({
		    where: {
		      email: 'kyle@test.com',
		    },
		    data: {
		      userPreference: {
		        connect: {
		          id: 'Acá iría el id de la preferencia',
		        },
		      },
		    },
		  });
		
		  console.log(user);
		}
      ```
   4. Verificamos la conexión con findFirst:
      ```ts 
      async function main() {
		  const user = await prisma.user.findFirst({
		    where: {
		      name: 'Kyle',
		    },
		    include: { userPreference: true },
		  });
		
		  console.log(user);
		}
      ```
  5. Probamos la desconexión del usuario con la preferencia:
     ```ts
     async function main() {
		  const user = await prisma.user.update({
		    where: {
		      email: 'kyle@test.com',
		    },
		    data: {
		      userPreference: {
		        disconnect: true,
		      },
		    },
		  });
		
		  console.log(user);
		}
     ```
     Con pasarle true es suficiente porque es una relación 1 a 1.
  Nota: Podemos hacer un connect cada vez que hacemos un create y podemos hacer disconnect cada vez que hacemos un update.
61. Ejemplo de `delete`:
    ```ts
    async function main() {
	  const user = await prisma.user.delete({
	    where: {
	      email: 'kyle@test.com',
	    }
	  });
	
	  console.log(user);
	}
    ```
    Ahora con `deleteMany`:
    ```ts
    async function main() {
	  const user = await prisma.user.deleteMany({
	    where: {
	      age: { gt: 20 },
	    },
	  });
	
	  console.log(user);
	}
    ```
   **Nota: Si al `deleteMany()` no le pasamos ningún parámetro, se borrará todo!**
   