# Guía de Integración: INGEDACA Secure Ops + Microsoft Graph (Excel en la Nube)

Esta guía documenta el paso a paso exacto para reemplazar el almacenamiento local (`localStorage`) de la aplicación de Gestión de Tareas de INGEDACA por una base de datos real utilizando un archivo de Excel alojado en el OneDrive o SharePoint corporativo de Ingedaca.

> [!NOTE]
> Esta arquitectura te permite seguir utilizando una aplicación frontend pura (HTML/JS) alojada en Vercel, delegando toda la base de datos, la seguridad y la autenticación a la infraestructura empresarial de Microsoft que ustedes ya pagan.

---

## FASE 1: Configurar la Base de Datos (Archivo Excel)

Para que Microsoft Graph reconozca tu Excel como una base de datos, los datos deben estar dentro de un formato de "Tabla".

1. **Crea el archivo:** Abre tu OneDrive de @ingedaca.com (o el SharePoint de la empresa) y crea un nuevo libro de Excel llamado `Ingedaca_Ops_DB.xlsx`.
2. **Crea los encabezados:** En la Hoja 1, crea las columnas: `ID_Proyecto`, `Tarea`, `Responsable`, `Prioridad`, `Estatus`, `Fecha_Limite`.
3. **Formatear como Tabla:**
   - Selecciona los encabezados y un par de filas en blanco debajo.
   - Ve a la pestaña **Insertar** > **Tabla**. (Asegúrate de marcar "La tabla tiene encabezados").
4. **Nombra la Tabla:** Ve a la pestaña de "Diseño de Tabla" y en el nombre de la tabla (arriba a la izquierda) ponle `TablaTareas` sin espacios.
5. **Copia el ID del Archivo:** Comparte o copia el enlace del archivo; más adelante usaremos Graph Explorer para obtener su ID único.

---

## FASE 2: Registrar la Aplicación en Microsoft Entra ID (Azure)

Este paso autoriza a tu aplicación (en Vercel) a solicitar inicios de sesión y acceder al Excel. Esto lo debes hacer con tu cuenta de administrador de Ingedaca.

1. Ve al portal de Azure: [portal.azure.com](https://portal.azure.com) e inicia sesión.
2. Busca el servicio **Microsoft Entra ID** (anteriormente Azure Active Directory).
3. En el menú izquierdo, ve a **Registros de aplicaciones (App registrations)** y dale a **Nuevo registro**.
4. **Nombra tu App:** Ponle `Ingedaca Secure Ops`.
5. **Tipos de cuentas:** Selecciona "Solo las cuentas de este directorio organizativo (INGEDACA)".
6. **URI de redirección:**
   - Selecciona la plataforma: **Aplicación de página única (SPA)**.
   - Coloca la URL donde probarás. P. ej.: `http://localhost:5500` para pruebas locales, o `https://ingedacatest.vercel.app/gestion-tareas/index.html` para Vercel. (Puedes agregar ambas).
7. Dale a **Registrar**.

> [!IMPORTANT]
> Una vez registrada, copia el **Id. de aplicación (cliente)** eb8e349c-e3ab-4cbb-a5a9-b78fd5651947

y el **Id. de directorio (inquilino)**
233c092f-a76d-482f-a19f-bfd2d29add33
. Lo necesitaremos para el código.

Identificador de objeto
:
9d4508d3-2ab4-440e-b65d-8e4e5d1366c0

---

## FASE 3: Permisos de la API (Microsoft Graph)

La aplicación ahora existe, pero debemos decirle a Microsoft que esta app puede leer y escribir archivos de Excel.

1. En el menú de tu aplicación recién creada en Entra ID, ve a **Permisos de API**.
2. Dale a **Agregar un permiso** > **Microsoft Graph** > **Permisos delegados**.
3. Busca y marca las siguientes casillas:
   - `User.Read` (Ya debería estar por defecto)
   - `Files.ReadWrite` (Para leer y escribir en tu Excel)
4. Haz clic en **Agregar permisos**.
5. *(Opcional pero recomendado)* Al lado de los permisos, haz clic en el botón **"Conceder consentimiento de administrador para INGEDACA"** para que a los usuarios no les salga el molesto cuadro de confirmación.

---

## FASE 4: Actualización del Código en `index.html`

Ahora que todo está configurado, inyectamos el código en la app.

### 1. Colocar tu Client ID real

Busca este bloque en el archivo `index.html` y reemplaza los datos:

```javascript
const msalConfig = {
    auth: {
        clientId: "PEGA_AQUÍ_EL_ID_DE_APLICACIÓN_CLIENTE",
        authority: "https://login.microsoftonline.com/PEGA_AQUÍ_EL_ID_DE_INQUILINO",
        redirectUri: window.location.origin
    }
};
```

### 2. Pedir un Token de Acceso (Access Token)

Justo después de verificar que el usuario está en la Whitelist, debes pedirle un código de acceso a Microsoft para que te deje tocar el Excel. 

```javascript
// Variable global para usar el token en nuestras consultas
let accessToken = ""; 

async function getAccessToken(){
    const request = {
        scopes: ["User.Read", "Files.ReadWrite"],
        account: msalInstance.getAllAccounts()[0]
    };
    
    try {
        // Pide el token silenciosamente
        const response = await msalInstance.acquireTokenSilent(request);
        accessToken = response.accessToken;
    } catch (error) {
        // Si falla (porque expiró), pide popup
        const response = await msalInstance.acquireTokenPopup(request);
        accessToken = response.accessToken;
    }
}
```

### 3. Leer las tareas del Excel (GET)

En lugar de usar el `localStorage` en tu función `loadData()`, harás una llamada (fetch) a Graph API.

```javascript
async function loadDataFromExcel() {
    await getAccessToken(); // Asegurarnos de tener el token
    
    // Necesitas el ItemID de tu archivo Excel. 
    // Lo consigues usando https://developer.microsoft.com/en-us/graph/graph-explorer
    const excelId = "TU_EXCEL_ITEM_ID"; 
    
    const endpoint = `https://graph.microsoft.com/v1.0/me/drive/items/${excelId}/workbook/tables/TablaTareas/rows`;

    const response = await fetch(endpoint, {
        method: "GET",
        headers: { "Authorization": `Bearer ${accessToken}` }
    });
    
    const data = await response.json();
    
    // Convertir las filas del Excel al formato de tu Array de 'tasks'
    tasks = data.value.map(row => {
        return {
            projectId: row.values[0][0], // Columna A
            task: row.values[0][1],      // Columna B
            owner: row.values[0][2],     // Columna C
            priority: row.values[0][3],  // Columna D
            status: row.values[0][4],    // Columna E
            dueDate: row.values[0][5]    // Columna F
        };
    });
    
    renderTasks();
}
```

### 4. Guardar tareas nuevas en el Excel (POST)

Cuando el gerente añada una tarea nueva, en lugar de usar `localStorage.setItem` usaremos un `POST` para agregar una fila en el Excel:

```javascript
async function saveNewTaskToExcel(newTask) {
    await getAccessToken();
    const excelId = "TU_EXCEL_ITEM_ID";
    const endpoint = `https://graph.microsoft.com/v1.0/me/drive/items/${excelId}/workbook/tables/TablaTareas/rows`;
    
    const fila = [
        [
            newTask.projectId,
            newTask.task,
            newTask.owner,
            newTask.priority,
            newTask.status,
            newTask.dueDate
        ]
    ];

    await fetch(endpoint, {
        method: "POST",
        headers: { 
            "Authorization": `Bearer ${accessToken}`,
            "Content-Type": "application/json"
        },
        body: JSON.stringify({ "values": fila })
    });
}
```

---

## 🚀 Resumen del Flujo de Trabajo
1. Subes el archivo `index.html` modificado a **Vercel**.
2. Abres Vercel, el sistema te protege con la pantalla de inicio.
3. Haces login con tu correo `@ingedaca.com` (apoyado por el doble factor estándar de tu empresa).
4. El sistema pide el *Access Token* y extrae las tareas que tengas en tu hoja de Ingedaca en OneDrive.
5. Trabajan y actualizan con normalidad, mientras todos los cambios se inyectan silenciosamente por código hacia ese Excel privado.
