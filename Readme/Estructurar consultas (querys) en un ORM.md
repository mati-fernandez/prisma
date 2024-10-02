#query #ORM #Prisma
# Cada vez que quiero hacer una consulta diferente a la BBDD debo modificar la función main del mismo script.ts?

No necesariamente. Aunque puedes hacer consultas directamente en la función main de tu script TypeScript, es mejor práctica estructurar tu código para que sea más modular y mantenible. Aquí te dejo algunas recomendaciones:
1. Funciones Separadas para Consultas

En lugar de tener todas las consultas dentro de main, puedes crear funciones separadas para cada consulta. Por ejemplo:

```typescript
async function getAllUsers() {
  const users = await prisma.user.findMany();
  return users;
}

async function getUserById(id: number) {
  const user = await prisma.user.findUnique({
    where: { id },
  });
  return user;
}

async function main() {
  const allUsers = await getAllUsers();
  console.log(allUsers);

  const user = await getUserById(1);
  console.log(user);
}

main().catch(e => {
  console.error(e);
}).finally(async () => {
  await prisma.$disconnect();
});
```

2. Usar Parámetros

Si tienes consultas similares que cambian solo en algunos parámetros, puedes hacer funciones que acepten esos parámetros.
3. Modularidad

Si tu proyecto crece, considera dividir tus funciones en archivos separados. Por ejemplo, podrías tener un archivo userService.ts donde manejes todas las funciones relacionadas con usuarios, y luego importarlas en tu archivo principal.
4. Usar un ORM

Si estás usando Prisma (que es un ORM), aprovecha sus características para estructurar tus consultas de manera que sea más fácil de leer y mantener.
Conclusión

Organizar tu código de esta manera no solo mejora la legibilidad, sino que también facilita el mantenimiento y la adición de nuevas consultas sin modificar la estructura base del script.

# pero en ese ejemplo al ejecutar el main está siempre haciendo esas mismas dos querys? 

Sí, en el ejemplo que te di, la función main siempre ejecuta las mismas dos consultas (todas las funciones que se llaman dentro de main). Si deseas tener más flexibilidad para ejecutar diferentes consultas sin modificar la función main cada vez, puedes hacerlo de varias maneras. Aquí hay algunas sugerencias:
1. Usar Parámetros en la Función main

Puedes pasar un parámetro a la función main para decidir qué consulta ejecutar:

```typescript
async function main(action: string) {
  if (action === 'getAllUsers') {
    const allUsers = await getAllUsers();
    console.log(allUsers);
  } else if (action === 'getUserById') {
    const user = await getUserById(1); // Cambia el ID según sea necesario
    console.log(user);
  } else {
    console.log('Acción no reconocida');
  }
}

const actionToRun = process.argv[2]; // Puedes pasar el tipo de acción como argumento al ejecutar el script
main(actionToRun).catch(e => {
  console.error(e);
}).finally(async () => {
  await prisma.$disconnect();
});
```

2. Usar un Switch o If-Else en main

Otra opción es usar una declaración switch para manejar diferentes acciones:

```typescript

async function main(action: string) {
  switch (action) {
    case 'getAllUsers':
      const allUsers = await getAllUsers();
      console.log(allUsers);
      break;
    case 'getUserById':
      const user = await getUserById(1); // Cambia el ID según sea necesario
      console.log(user);
      break;
    default:
      console.log('Acción no reconocida');
  }
}

// Ejecutar el script con el argumento deseado
const actionToRun = process.argv[2]; // Puedes pasar el tipo de acción como argumento al ejecutar el script
main(actionToRun).catch(e => {
  console.error(e);
}).finally(async () => {
  await prisma.$disconnect();
});
```

3. Ejecutar desde la Línea de Comandos

Con el enfoque anterior, podrías ejecutar tu script desde la terminal pasando un argumento para indicar qué consulta deseas ejecutar. Por ejemplo:

```bash
ts-node script.ts getAllUsers
```

4. Usar Interacción con el Usuario

Si prefieres, podrías agregar interacción con el usuario para elegir qué consulta ejecutar, aunque esto requiere un poco más de complejidad.
Conclusión

De esta manera, no necesitas modificar la función main cada vez que quieras hacer una consulta diferente. Simplemente puedes pasar un argumento o usar alguna lógica para decidir qué consulta ejecutar en función de las necesidades.