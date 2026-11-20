import sqlite3

# 1 - CREAR LA BASE DE DATOS CON EL NOMBRE ARINC-429-7A
conn = sqlite3.connect("ARINC-429-7A.db")
conn.execute("PRAGMA foreign_keys = ON;")
cursor = conn.cursor()

# Limpiar tablas previas si el script se vuelve a ejecutar
cursor.executescript("""
DROP TABLE IF EXISTS rol_permiso;
DROP TABLE IF EXISTS usuario_rol;
DROP TABLE IF EXISTS permisos;
DROP TABLE IF EXISTS roles;
DROP TABLE IF EXISTS usuarios;
""")

# 2 - CREAR LA ESTRUCTURA DE LA BASE DE DATOS
cursor.executescript("""
CREATE TABLE usuarios (
    id_usuario INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre TEXT NOT NULL,
    email TEXT NOT NULL UNIQUE,
    password TEXT NOT NULL,
    activo INTEGER DEFAULT 1,
    fecha_creacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE roles (
    id_rol INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre_rol TEXT NOT NULL UNIQUE,
    descripcion TEXT
);

CREATE TABLE permisos (
    id_permiso INTEGER PRIMARY KEY AUTOINCREMENT,
    nombre_permiso TEXT NOT NULL UNIQUE,
    descripcion TEXT
);

CREATE TABLE usuario_rol (
    id_usuario INTEGER NOT NULL,
    id_rol INTEGER NOT NULL,
    fecha_asignacion TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (id_usuario, id_rol),
    FOREIGN KEY (id_usuario) REFERENCES usuarios(id_usuario) ON DELETE CASCADE,
    FOREIGN KEY (id_rol) REFERENCES roles(id_rol) ON DELETE CASCADE
);

CREATE TABLE rol_permiso (
    id_rol INTEGER NOT NULL,
    id_permiso INTEGER NOT NULL,
    PRIMARY KEY (id_rol, id_permiso),
    FOREIGN KEY (id_rol) REFERENCES roles(id_rol) ON DELETE CASCADE,
    FOREIGN KEY (id_permiso) REFERENCES permisos(id_permiso) ON DELETE CASCADE
);
""")

# 3 - CARGAR LOS DATOS EN CADA UNA DE LAS TABLAS
usuarios = [
    ("Valentino Fernández", "valentino@arinc429.com", "hash_pass_123"),
    ("Julián Battaglia", "julian@arinc429.com", "hash_pass_456"),
    ("Maia Sosa", "maia@arinc429.com", "hash_pass_789"),
    ("Benjamín Cornejo", "benjamin@arinc429.com", "hash_pass_000"),
]
cursor.executemany(
    "INSERT INTO usuarios (nombre, email, password) VALUES (?, ?, ?);", usuarios
)

roles = [
    ("Administrador", "Control total del sistema y gestión de usuarios"),
    ("Técnico Aviónico", "Configuración de buses ARINC 429, transmisión y diagnóstico"),
    ("Operador / Lectura", "Visualización de tramas y monitoreo en tiempo real"),
]
cursor.executemany(
    "INSERT INTO roles (nombre_rol, descripcion) VALUES (?, ?);", roles
)

permisos = [
    ("GESTIONAR_USUARIOS", "Crear, editar y eliminar usuarios y roles"),
    ("TRANSMITIR_TRAMAS", "Enviar palabras de 32 bits por puerto serie / bus ARINC"),
    ("CONFIGURAR_INSTRUMENTOS", "Dar de alta y calibrar rangos min/max de instrumentos"),
    ("VER_LOGS", "Consultar historial de envíos y lecturas del bus"),
]
cursor.executemany(
    "INSERT INTO permisos (nombre_permiso, descripcion) VALUES (?, ?);", permisos
)

usuario_rol = [
    (1, 1),  # Valentino -> Administrador
    (1, 2),  # Valentino -> Técnico Aviónico
    (2, 2),  # Julián -> Técnico Aviónico
    (3, 2),  # Maia -> Técnico Aviónico
    (4, 3),  # Benjamín -> Operador / Lectura
]
cursor.executemany(
    "INSERT INTO usuario_rol (id_usuario, id_rol) VALUES (?, ?);", usuario_rol
)

rol_permiso = [
    (1, 1), (1, 2), (1, 3), (1, 4),  # Administrador
    (2, 2), (2, 3), (2, 4),          # Técnico Aviónico
    (3, 4),                          # Operador / Lectura
]
cursor.executemany(
    "INSERT INTO rol_permiso (id_rol, id_permiso) VALUES (?, ?);", rol_permiso
)

conn.commit()


def imprimir_tabla(titulo, columnas, filas):
    print(f"\n{'=' * 75}\n{titulo}\n{'=' * 75}")
    anchos = [
        max(len(str(col)), *(len(str(fila[i])) for fila in filas))
        for i, col in enumerate(columnas)
    ]
    encabezado = " | ".join(col.ljust(anchos[i]) for i, col in enumerate(columnas))
    print(encabezado)
    print("-" * len(encabezado))
    for fila in filas:
        print(" | ".join(str(val).ljust(anchos[i]) for i, val in enumerate(fila)))


# 3 (bis) - TODOS LOS USUARIOS, ROLES Y PERMISOS (INNER JOIN)
cursor.execute("""
SELECT 
    u.id_usuario,
    u.nombre AS usuario,
    u.email,
    r.nombre_rol AS rol,
    p.nombre_permiso AS permiso
FROM usuarios u
INNER JOIN usuario_rol ur ON u.id_usuario = ur.id_usuario
INNER JOIN roles r ON ur.id_rol = r.id_rol
INNER JOIN rol_permiso rp ON r.id_rol = rp.id_rol
INNER JOIN permisos p ON rp.id_permiso = p.id_permiso
ORDER BY u.id_usuario, r.id_rol, p.id_permiso;
""")
imprimir_tabla(
    "3 - TODOS LOS USUARIOS, ROLES Y PERMISOS",
    ["ID", "Usuario", "Email", "Rol", "Permiso"],
    cursor.fetchall(),
)

# 4 - TODOS LOS ROLES Y PERMISOS (INNER JOIN)
cursor.execute("""
SELECT 
    r.id_rol,
    r.nombre_rol AS rol,
    p.nombre_permiso AS permiso,
    p.descripcion AS detalle
FROM roles r
INNER JOIN rol_permiso rp ON r.id_rol = rp.id_rol
INNER JOIN permisos p ON rp.id_permiso = p.id_permiso
ORDER BY r.id_rol, p.id_permiso;
""")
imprimir_tabla(
    "4 - TODOS LOS ROLES Y PERMISOS",
    ["ID Rol", "Rol", "Permiso", "Descripción"],
    cursor.fetchall(),
)

# 5 - TODOS LOS USUARIOS CON TODOS LOS ROLES (INNER JOIN)
cursor.execute("""
SELECT 
    u.id_usuario,
    u.nombre AS usuario,
    u.email,
    r.id_rol,
    r.nombre_rol AS rol
FROM usuarios u
INNER JOIN usuario_rol ur ON u.id_usuario = ur.id_usuario
INNER JOIN roles r ON ur.id_rol = r.id_rol
ORDER BY u.id_usuario, r.id_rol;
""")
imprimir_tabla(
    "5 - TODOS LOS USUARIOS CON TODOS LOS ROLES",
    ["ID Usuario", "Usuario", "Email", "ID Rol", "Rol"],
    cursor.fetchall(),
)

conn.close()
