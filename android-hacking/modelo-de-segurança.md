# Modelo de Segurança do Android

O Android implementa múltiplas camadas de defesa independentes. Entender cada uma é necessário porque a maior parte do trabalho em exploits modernos consiste em contorná-las.

## Application Sandbox e UIDs

O Android isola apps usando UIDs do Linux: o App A (UID 10001) não pode ler arquivos do App B (UID 10002).

**Objetivo do ataque:** obter RCE dentro do contexto do app alvo para herdar seu UID e acessar seus dados privados em `/data/data/com.alvo/`.

## Permissões e Scoped Storage

Até o Android 9, a permissão `READ_EXTERNAL_STORAGE` concedia acesso irrestrito ao armazenamento externo. A partir do **Android 10**, o Scoped Storage limita cada app a ver apenas seus próprios arquivos e mídias públicas, sem acesso à pasta de downloads ou arquivos de outros apps.

**Impacto:** Path Traversal e Directory Listing ficaram mais difíceis. O foco passa a ser FileProviders mal configurados, que ainda podem permitir acesso indevido a arquivos do sistema.

## Code Signing e Repackaging

Todo APK precisa ser assinado. Os esquemas de assinatura existentes são:

| Esquema | Cobertura |
|---|---|
| v1 (JAR Signing) | Verifica arquivos individuais |
| v2 / v3 / v4 | Verifica o APK inteiro bit a bit |

**Obstáculo:** modificar e recompilar um APK quebra a assinatura v2/v3, e o Android rejeita a instalação.

**Solução:** remover a assinatura original (`META-INF`), aplicar as modificações e re-assinar com chave própria via `apksigner`. A troca de assinatura implica perda dos dados locais do app e quebra integrações que validam o hash do certificado (ex: Google Maps, Facebook Login).

## SELinux

O SELinux opera em modo MAC (Mandatory Access Control): bloqueia ações com base em políticas, independentemente de quem executa.

**Cenário real:** mesmo com root, o SELinux pode impedir que um shell reverso acesse a câmera ou injete código em outro processo.

**Bypass em ambiente de teste:**
```bash
setenforce 0  # coloca o SELinux em modo permissivo
```

Em exploits reais, é necessário encontrar falhas no kernel para desativar a política em tempo de execução.

## Flag de Depuração

Verifique sempre o `AndroidManifest.xml`. Se `android:debuggable="true"` estiver presente, é possível conectar via JDWP (Java Debug Wire Protocol) e obter um shell dentro do app sem nenhum exploit.

```bash
adb jdwp              # lista processos com JDWP ativo
adb forward tcp:8700 jdwp:<pid>
jdb -attach localhost:8700
```
