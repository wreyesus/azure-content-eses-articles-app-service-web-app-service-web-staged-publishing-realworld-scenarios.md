<properties
	pageTitle="Ediciones de Azure Active Directory | Microsoft Azure"
	description="Un tema que explica las opciones de las ediciones gratuitas y de pago de Azure Active Directory. Azure Active Directory Basic, Azure Active Directory Premium P1 y Azure Active Directory Premium P2 son las ediciones de pago."
	services="active-directory"
	documentationCenter=""
	authors="curtand"
	manager="femila"
	editor=""/>

<tags
	ms.service="active-directory"
	ms.workload="identity"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/09/2016"
	ms.author="curtand"/>

# Ediciones de Azure Active Directory

Todos los servicios de negocios de Microsoft Online dependen de Azure Active Directory (Azure AD) para el inicio de sesión y otras necesidades de identidad. Si se suscribe a alguno de los servicios de negocios de Microsoft Online (por ejemplo, Office 365, Microsoft Azure, etc.), recibirá Azure AD con acceso a todas las características gratuitas que se describen a continuación.

Azure Active Directory es un servicio que proporciona funcionalidades completas de identidad y acceso en la nube para empleados, asociados y clientes. En él se combinan servicios de directorio, regulación avanzada de identidades, una completa plataforma basada en estándares para los desarrolladores y la administración del acceso tanto para sus propias aplicaciones como para otras miles de aplicaciones previamente integradas. Con la edición gratuita de Azure Active Directory puede administrar usuarios y grupos, sincronizar directorios locales, obtener inicio de sesión único entre Azure, Office 365 y miles de aplicaciones SaaS conocidas como Salesforce, Workday, Concur, DocuSign, Google Apps, Box, ServiceNow, Dropbox, etc. Para obtener más información sobre Azure Active Directory, lea [¿Qué es AD?](active-directory-whatis.md).

Para mejorar su instancia de Azure Active Directory, puede agregar funcionalidades de pago con las ediciones Azure Active Directory Basic, Premium P1 y Premium P2. Las ediciones de pago de Azure Active Directory se crean encima del directorio gratuito existente, y proporcionan funcionalidades de tipo empresarial que abarcan autoservicio, supervisión mejorada, informes de seguridad, Multi-Factor Authentication (MFA) y un acceso seguro para sus trabajadores móviles.

Las suscripciones a Office 365 incluyen características adicionales de Azure Active Directory que se describen en la siguiente tabla de comparación.


> [AZURE.NOTE] Para ver las opciones de precios de estas ediciones, consulte [Precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/). Azure Active Directory Premium P1, Premium P2 y Azure Active Directory Basic no se admiten actualmente en China. Póngase en contacto con nosotros en el Foro de Azure Active Directory para más información.


- **Edición básica de Azure Active Directory**: esta edición, concebida para los trabajadores de tareas con primeras necesidades en la nube, proporciona soluciones de administración de identidades de autoservicio y de acceso a las aplicaciones basado en la nube. Con la edición básica de Azure Active Directory, obtiene características de mejora de la productividad y reducción de costos, como administración de acceso basado en grupo, autoservicio de restablecimiento de contraseña para aplicaciones en la nube y proxy de aplicaciones de Azure Active Directory (para publicar aplicaciones web locales con Azure Active Directory), y todas ellas respaldadas por un contrato de nivel de servicio empresarial con un tiempo de actividad del 99,9 por ciento.

- **Azure Active Directory Premium P1**: esta edición, dirigida a las organizaciones con necesidades más acuciantes de administración de identidades y acceso, agrega completas funcionalidades de administración de identidades de tipo empresarial y permite a los usuarios híbridos acceder sin problemas a funcionalidades locales y en la nube. Incluye todo lo que necesitan los trabajadores de información y los administradores de identidades en los entornos híbridos para el acceso entre aplicaciones, la administración de identidades y acceso de autoservicio (IAM), la protección de identidades y la seguridad en la nube. Admite recursos avanzados de administración y delegación, como grupos dinámicos y administración de grupos de autoservicio. Incluye Microsoft Identity Manager (un conjunto de aplicaciones de administración de identidades y acceso locales) y proporciona funcionalidades de escritura diferida en la nube que permiten soluciones como el autoservicio de restablecimiento de contraseña para los usuarios locales.

- **Azure Active Directory Premium P2**: esta nueva oferta, diseñada con protección avanzada para todos los usuarios y administradores, incluye todas las funcionalidades de Azure AD Premium P1, así como nuestras nuevas características Identity Protection y Privileged Identity Management. Azure Active Directory Identity Protection aprovecha miles de millones de señales para proporcionar acceso condicional basado en el riesgo a las aplicaciones y los datos críticos de una empresa. También le ayudamos a administrar y proteger cuentas privilegiadas con Azure Active Directory Privileged Identity Management para que pueda descubrir, restringir y supervisar a los administradores y su acceso a los recursos y proporcionar acceso a tiempo cuando sea necesario.

Para suscribirse y empezar a usar Active Directory Premium hoy, vea [Introducción a Azure Active Directory Premium](active-directory-get-started-premium.md).


> [AZURE.NOTE]
Hay varias funcionalidades de Azure Active Directory disponibles mediante las ediciones de "pago por uso":
>
>- Active Directory B2C es la solución de administración de identidades y acceso para las aplicaciones orientados al consumidor. Para obtener más información, consulte [Azure Active Directory B2C](https://azure.microsoft.com/documentation/services/active-directory-b2c/)

>-	Azure Multi-Factor Authentication se puede usar a través de proveedores por usuario o por autenticación. Para obtener más detalles, consulte [Qué es Azure Multi-Factor Authentication](../multi-factor-authentication/multi-factor-authentication.md).


##Comparación de características generalmente disponibles

> [AZURE.NOTE] Para obtener una vista diferente de estos datos, consulte las [funcionalidades de Azure Active Directory](https://www.microsoft.com/en/server-cloud/products/azure-active-directory/features.aspx).




**Características comunes**

- [Objetos de directorio](#directory-objects)

- [Administración de usuarios y grupos (agregar/actualizar/eliminar)/aprovisionamiento basado en el usuario, registro de dispositivos](#usergroup-management-addupdatedelete-user-based-provisioning-device-registration)

- [Inicio de sesión único (SSO)](#single-sign-on-sso)

- [Cambio de contraseña de autoservicio para usuarios en la nube](#self-service-password-change-for-cloud-users)

- [Conexión (motor de sincronización que extiende los directorios locales a Azure Active Directory)](#connect-sync-engine-that-extends-on-premises-directories-to-azure-active-directory)

- [Seguridad/informes de uso](#securityusage-reports)



**Características de la edición Basic**

- [Aprovisionamiento y administración del acceso basados en grupo](#group-based-access-managementprovisioning)

- [Restablecimiento de contraseña de autoservicio para usuarios en la nube](#self-service-password-reset-for-cloud-users)

- [Personalización de marca de la compañía (personalización de las páginas de inicio de sesión y del panel de acceso)](#company-branding-logon-pagesaccess-panel-customization)

- [Proxy de aplicación](#application-proxy)

- [Contrato de nivel de servicio del 99,9 %](#sla-999)


**Características de la edición Premium P1**

- [Administración de grupos y aplicaciones de autoservicio/incorporaciones de aplicaciones de autoservicio/grupos dinámicos](#self-service-group-and-app-managementself-service-application-additions-dynamic-groups)

- [Restablecimiento de contraseñas de autoservicio/cambio/desbloqueo con escritura diferida local](#self-service-password-resetchangeunlock-with-on-premises-write-back)

- [Multi-Factor Authentication (en la nube y local (servidor MFA))](#multi-factor-authentication-cloud-and-on-premises-mfa-server)

- [CAL de MIM + servidor MIM](#mim-cal-mim-server)

- [Detección de aplicaciones de nube](#cloud-app-discovery)

- [Connect Health](#connect-health)

- [Sustitución automática de la contraseña para cuentas de grupo](#automatic-password-rollover-for-group-accounts)

**Características de la edición Premium P2**

- [Identity Protection](active-directory-identityprotection.md)

- [Privileged Identity Management](active-directory-privileged-identity-management-configure.md)

**Azure Active Directory Join, solo características relacionadas con Windows 10**

- [Conectar un dispositivo a Azure AD, SSO de escritorio, Microsoft Passport para Azure AD, recuperación de Bitlocker de administrador](#join-a-device-to-azure-ad-desktop-sso-microsoft-passport-for-azure-ad-administrator-bitlocker-recovery)

- [Inscripción automática de MDM, recuperación de Bitlocker de autoservicio, administradores locales adicionales para dispositivos con Windows 10 a través de Azure AD Join](#mdm-auto-enrolment-self-service-bitlocker-recovery-additional-local-administrators-to-windows-10-devices-via-azure-ad-join)


## Características comunes
#### Objetos de directorio

**Tipo:** características comunes

La cuota de uso predeterminada es de 150.000 objetos. Un objeto es una entrada del servicio de directorio que está representada por un nombre completo único. Un ejemplo de objeto sería una entrada de usuario empleada para la autenticación. Si necesita sobrepasar la cuota predeterminada, póngase en contacto con el servicio de soporte técnico. El límite de 500 000 objetos no es aplicable a Office 365, Microsoft Intune ni ningún otro servicio en línea de pago de Microsoft que se base en Azure Active Directory para los servicios de directorio.


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| Hasta 500.000 objetos| Sin límite de objetos| Sin límite de objetos| Sin límite de objetos para las cuentas de usuario de Office 365|



#### Administración de usuarios y grupos (agregar/actualizar/eliminar)/aprovisionamiento basado en el usuario, registro de dispositivos

**Tipo:** características comunes

**Disponibilidad:**


| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|

**Más detalles:**

- [Administración del directorio de Azure AD](active-directory-administer.md)
- [Información general sobre el Registro de dispositivos de Azure Active Directory](active-directory-conditional-access-device-registration-overview.md)




#### Inicio de sesión único (SSO)

**Tipo:** Características comunes


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| 10 aplicaciones por usuario (1) | 10 aplicaciones por usuario (1) | Ilimitado (2) | 10 aplicaciones por usuario (1)|

1. Con las ediciones Gratis y Básico de Azure AD, los usuarios finales a los que se haya asignado acceso a aplicaciones de SaaS pueden ver hasta 10 aplicaciones en su Panel de acceso y obtener inicio de sesión único en ellas. Los usuarios pueden configurar el inicio de sesión único y asignar acceso de usuario a tantas aplicaciones de SaaS como deseen con los niveles Gratis y Básico; sin embargo, los usuarios finales verán solo 10 aplicaciones en su Panel de acceso a la vez.

2. Integración de autoservicio de cualquier aplicación que admita SAML, SCIM o autenticación basada en formularios mediante el uso de plantillas proporcionadas en el menú de la Galería de aplicaciones. Para obtener más información, consulte [Configuración del inicio de sesión único en aplicaciones que no están en la Galería de aplicaciones de Azure Active Directory](active-directory-saas-custom-apps.md).

**Más detalles:**

- [Administración de aplicaciones con Azure Active Directory (AD)](active-directory-enable-sso-scenario.md)



#### Cambio de contraseña de autoservicio para usuarios en la nube

**Tipo:** características comunes

**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|

**Más detalles:**

- [Actualización de la propia contraseña](active-directory-passwords-update-your-own-password.md)




#### Conexión (motor de sincronización que extiende los directorios locales a Azure Active Directory)

**Tipo:** características comunes


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|

**Más detalles:**

- [Integración de las identidades locales con Azure Active Directory](active-directory-aadconnect.md)



#### Informes de seguridad y uso

**Tipo:** características comunes


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| 3 informes básicos| 3 informes básicos| Informes avanzados| 3 informes básicos|

**Más detalles:**

- [Visualización de los informes de acceso y uso](active-directory-view-access-usage-reports.md)




## Características de las ediciones Premium y Basic
#### Aprovisionamiento y administración del acceso basados en grupo

**Tipo:** características básicas


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | ![Comprobar][12]| ![Comprobar][12]| |

**Más detalles:**

- [Uso de un grupo para administrar el acceso a las aplicaciones SaaS](active-directory-accessmanagement-group-saasapps.md)



#### Restablecimiento de contraseña de autoservicio para usuarios en la nube

**Tipo:** características básicas


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|

**Más detalles:**

- [Restablecimiento de contraseña de Azure AD para usuarios y administradores](active-directory-passwords.md)



#### Personalización de marca de la compañía (personalización de las páginas de inicio de sesión y del panel de acceso)

**Tipo:** características básicas


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|

**Más detalles:**

- [Incorporación de la marca de empresa a sus páginas de inicio de sesión y panel de acceso](active-directory-add-company-branding.md)



#### Proxy de aplicación

**Tipo:** características básicas


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | ![Comprobar][12]| ![Comprobar][12]| |

**Más detalles:**

- [Provisión de acceso remoto seguro a aplicaciones locales](active-directory-application-proxy-get-started.md)



#### Contrato de nivel de servicio del 99,9 %

**Tipo:** características básicas


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|

**Más detalles:**

- [Contratos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/)




## Características de la edición Premium
#### Administración de grupos y aplicaciones de autoservicio/incorporaciones de aplicaciones de autoservicio/grupos dinámicos

**Tipo:** Características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |





#### Restablecimiento de contraseñas de autoservicio/cambio/desbloqueo con escritura diferida local

**Tipo:** características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |





#### Multi-Factor Authentication (en la nube y local (servidor MFA))

**Tipo:** características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| Se limita a la nube solo para aplicaciones de Office 365|

**Más detalles:**

- [¿Qué es Azure Multi-Factor Authentication?](../multi-factor-authentication/multi-factor-authentication.md)



#### CAL de MIM + servidor MIM

Con las licencias de Windows Server (cualquier edición), se conceden derechos de software de servidor de Microsoft Identity Manager. Puesto que Microsoft Identity Manager se ejecuta en el sistema operativo Windows Server, siempre que un servidor ejecute una copia válida y con licencia de Windows Server, Microsoft Identity Manager podrá instalarse y usarse en ese servidor. No se necesita ninguna otra licencia de servidor de Microsoft Identity Manager.

**Tipo:** Características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |





#### Cloud App Discovery

**Tipo:** Características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |

**Más detalles:**

- [Búsqueda de aplicaciones de nube no administradas con Cloud App Discovery](active-directory-cloudappdiscovery-whatis.md)



#### Azure AD Connect Health

**Tipo:** Características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |

**Más detalles:**

- [Supervisión de la infraestructura de identidad local y los servicios de sincronización en la nube](active-directory-aadconnect-health.md)



#### Sustitución automática de la contraseña para cuentas de grupo

**Tipo:** características de la edición Premium


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |


#### Identity Protection

**Tipo:** Características de la edición Premium

| Edición gratuita| Edición básica| Edición Premium P2 | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |


#### Privileged Identity Management

**Tipo:** Características de la edición Premium

| Edición gratuita| Edición básica| Edición Premium P2 | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |


## Azure Active Directory Join, solo características relacionadas con Windows 10
#### Conectar un dispositivo a Azure AD, SSO de escritorio, Microsoft Passport para Azure AD, recuperación de Bitlocker de administrador

**Tipo:** Azure Active Directory Join, solo características de Windows 10


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]| ![Comprobar][12]|




#### Inscripción automática de MDM, recuperación de Bitlocker de autoservicio, administradores locales adicionales para dispositivos con Windows 10 a través de Azure AD Join

**Tipo:** Azure Active Directory Join, solo características de Windows 10


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |


#### Enterprise State Roaming

**Tipo:** Azure Active Directory Join, solo características de Windows 10


**Disponibilidad:**

| Edición gratuita| Edición básica| Ediciones Premium (P1 y P2) | Solo aplicaciones de Office 365 |
| :-: | :-: | :-: | :-: |
| | | ![Comprobar][12]| |

**Más detalles:**

- [Enterprise State Roaming](active-directory-windows-enterprise-state-roaming-overview.md)


## Características de vista previa de Azure AD
Además de las características disponibles normalmente en las ediciones Free, Basic y Premium (P1 y P2), Azure AD también proporciona una colección de características de versión preliminar. Puede usar las características de versiones preliminares para obtener una impresión de las novedades futuras y determinar si estas características pueden ayudar a mejorar su entorno.

**Características de versiones preliminares disponibles:**

- [Colaboración B2B](active-directory-b2b-collaboration-overview.md)
- [Unidades administrativas](active-directory-administrative-units-management.md)
- [Integración de aplicaciones de HR](active-directory-saas-workday-inbound-tutorial.md)
- [Autenticación basada en certificados en iOS](active-directory-certificate-based-authentication-ios.md)
- [Autenticación basada en certificados en Android](active-directory-certificate-based-authentication-android.md)






## Pasos siguientes

- [Introducción a Azure Active Directory Premium](active-directory-get-started-premium.md)
- [Incorporación de la marca de empresa a sus páginas de inicio de sesión y panel de acceso](active-directory-add-company-branding.md)
- [Visualización de los informes de acceso y uso](active-directory-view-access-usage-reports.md)

<!--Image references-->
[12]: ./media/active-directory-editions/ic195031.png

<!---HONumber=AcomDC_0914_2016-->