# Mini-research — Laboratorio 02

**Tema elegido:** *(uno)*

- [ ] **A.** Modos de operación de cifrado por bloques (ECB vs CBC vs GCM) y por
  qué ECB filtra estructura (el "pingüino" de Adobe).
- [ ] **B.** HMAC y el ataque de length-extension: cómo funciona el ataque y por
  qué HMAC lo previene.
- [ ] **C.** Derivación de claves desde contraseñas (PBKDF2, bcrypt, scrypt,
  Argon2): por qué un `sha256(password)` no alcanza.
- [x] **D.** Cifrado asimétrico y firmas digitales: qué problema resuelven que el
  simétrico no.

## Desarrollo
### El problema del cifrado simétrico
En el cifrado simétrico, emisor y receptor usan **la misma clave** tanto para cifrar como para descifrar. Esto genera dos problemas centrales:

1. **Distribución de claves**: antes de comunicarse de forma segura, ambas partes necesitan compartir la clave secreta por algún canal. Pero si ese canal no es seguro, un atacante puede interceptarla. Es un problema circular: para comunicarte de forma segura necesitás un canal seguro previo.
2. **Escalabilidad**: en una red de *n* usuarios donde todos quieren hablar entre sí de forma privada, se necesitan n(n−1)/2 claves distintas. Con 100 usuarios ya son casi 5000 claves para gestionar.
3. **No hay autenticación de origen ni no repudio**: como ambas partes conocen la misma clave, cualquiera de las dos podría haber generado un mensaje dado. No hay forma de probar matemáticamente "quién" lo escribió, ni de impedir que alguien niegue haberlo enviado.

### Cómo lo resuelve el cifrado asimétrico
El cifrado asimétrico usa un **par de claves matemáticamente relacionadas**: una pública (se distribuye libremente) y una privada (se mantiene secreta). Lo que se cifra con una solo se descifra con la otra.

- **Elimina el problema de distribución**: la clave pública puede publicarse sin ningún riesgo, porque conocerla no permite descifrar mensajes ni suplantar al dueño de la clave privada. Este esquema resuelve el problema de distribución usando un par de claves matemáticamente vinculadas: una pública que cualquiera puede usar para cifrar datos, y una privada que solo el dueño posee y usa para descifrar. Esto elimina la necesidad de compartir un secreto a través del canal de comunicación.

- **Habilita las firmas digitales**: invirtiendo el uso de las claves, el emisor puede cifrar (firmar) con su clave privada, y cualquiera puede verificar con su clave pública que ese mensaje solo pudo haber sido generado por esa persona. Para firmar un mensaje M, Alicia lo cifra con su propia clave privada; el receptor puede descifrar esa firma con la clave pública de Alicia, y si el resultado coincide con el mensaje original, la firma es válida, ya que solo quien posee la clave privada de Alicia puede haberla generado. Esto da **integridad** (si el mensaje cambia, la firma no valida), **autenticación de origen** y **no repudio** (Alicia no puede negar haber firmado, porque solo ella tiene esa clave privada).

### Por qué no reemplaza al simétrico, sino que lo complementa
El cifrado asimétrico es mucho más lento y costoso computacionalmente que el simétrico para grandes volúmenes de datos. Los algoritmos de cifrado asimétrico son mucho más lentos que el cifrado simétrico, especialmente para grandes volúmenes de datos, y requieren más poder de procesamiento y memoria. Por eso, en la práctica, la mayoría de los sistemas reales (como TLS/HTTPS) usan un **esquema híbrido**: el cifrado asimétrico se usa solo para intercambiar de forma segura una clave simétrica efímera, y esa clave simétrica es la que después cifra el volumen real de datos de la sesión. La mayoría de los sistemas modernos usan un enfoque híbrido: cifrado asimétrico para intercambiar una clave simétrica, que luego cifra los datos masivos.

## Fuentes
1. Entro Security — *Symmetric vs. Asymmetric Encryption*: https://entro.security/glossary/symmetric-vs-asymmetric-encryption/
2. testRigor — *Cryptographic Algorithms: Symmetric vs. Asymmetric*: https://testrigor.com/blog/cryptographic-algorithms-symmetric-vs-asymmetric/
3. GeeksforGeeks — *Symmetric vs Asymmetric Key Encryption*: https://www.geeksforgeeks.org/computer-networks/difference-between-symmetric-and-asymmetric-key-encryption/
4. Keyfactor — *Symmetric vs Asymmetric Cryptography: Key Differences Explained*: https://www.keyfactor.com/blog/symmetric-vs-asymmetric-cryptography-whats-the-difference-and-when-to-use-each/

## Reflexión (3–5 líneas)
Este trabajo evidencia que la seguridad informática no depende de un mecanismo único, sino de combinar técnicas según el problema que cada una resuelve mejor. El cifrado híbrido muestra que la solución más robusta no es la más "fuerte" en abstracto, sino la mejor adaptada al contexto, equilibrando seguridad y eficiencia.