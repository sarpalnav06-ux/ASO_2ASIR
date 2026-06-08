# Active Directory – Proyecto 2 ASIR 2025/2026

## Dominio

`proyecto2.local`

## Estructura de Unidades Organizativas

```
proyecto2.local
└── OU=Proyecto2
    ├── OU=Usuarios
    ├── OU=Tecnicos
    ├── OU=Administradores
    └── OU=Grupos
        ├── GRP_Usuarios
        ├── GRP_Tecnicos
        └── GRP_Admins
```

## Grupos y roles

| Grupo | Rol en la aplicación | Permisos |
|-------|---------------------|----------|
| GRP_Admins | Administrador | Acceso total |
| GRP_Tecnicos | Técnico | Gestión de incidencias |
| GRP_Usuarios | Usuario | Gestión de tareas |

## Usuarios de prueba

| Usuario | OU | Grupo |
|---------|----|-------|
| admin | OU=Administradores | GRP_Admins |
| tecnico1 | OU=Tecnicos | GRP_Tecnicos |
| usuario1 | OU=Usuarios | GRP_Usuarios |

---

*Proyecto Intermodular 2º ASIR – 2025/2026*
