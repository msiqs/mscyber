# Engenharia Reversa de APK

A engenharia reversa permite analisar o código de um app sem executá-lo, revelando lógica de negócio, segredos hardcoded e superfície de ataque. A ferramenta central para isso é o **JADX**.

## Anatomia de um APK

Um `.apk` é um arquivo ZIP renomeado. Os arquivos relevantes para análise:

| Arquivo | Conteúdo |
|---|---|
| `AndroidManifest.xml` | Declaração de componentes e permissões (binário no APK, o JADX decodifica) |
| `classes.dex` | Código Java/Kotlin compilado em bytecode Dalvik |
| `lib/` | Bibliotecas nativas `.so` em C/C++ para ARM, x86 etc. |
| `res/values/strings.xml` | Strings de texto, frequentemente com chaves e URLs hardcoded |

## JADX

O JADX descompila o `classes.dex` e reconstrói o código Java em uma interface navegável, similar a uma IDE, com busca global, navegação por classes e rastreamento de variáveis.

```bash
# Abrir APK na interface gráfica
jadx-gui app.apk
```

## Checklist de Análise Estática

O objetivo não é ler o código linha por linha, mas buscar por padrões específicos que indicam vulnerabilidades.

### AndroidManifest.xml

Primeira parada obrigatória. Buscar por:
- `exported="true"`: componentes acessíveis por outros apps sem autenticação
- `android:allowBackup="true"`: backup do sandbox habilitado
- `android:debuggable="true"`: app com modo debug ativo em produção

### Segredos Hardcoded

Buscar pelas strings:

```
api_key, Authorization, Bearer, AWS, firebase, password, secret, aes, private_key
```

Verificar também `res/values/strings.xml`, onde chaves costumam ser armazenadas fora do código Java.

### Lógica de Criptografia

Buscar classes que importam `javax.crypto` ou `java.security`. O alvo são chaves simétricas (AES) com Key ou IV estáticos ou previsíveis.

### Endpoints de API

Buscar por `https://`, `/api/`, `/v1/` para mapear rotas não visíveis durante o uso normal: painéis admin, rotas de teste, endpoints legados.

## Java vs. Smali

O JADX exibe Java, que é uma reconstrução aproximada do código original. Para **modificar** o app (ex: remover verificações de segurança), é necessário editar o **Smali**, a representação legível do bytecode DEX.

Comparação:

```java
// Java (alto nível, lido no JADX)
if (senha.equals("admin")) { ... }
```

```smali
# Smali (baixo nível, editado com Apktool)
invoke-virtual {v0, v1}, Ljava/lang/String;->equals(Ljava/lang/Object;)Z
if-eqz v2, :cond_0
```

> Para modificar APKs, a ferramenta utilizada é o `Apktool`. A análise estática no JADX é somente leitura.
