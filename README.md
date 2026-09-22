# X05 xCloud Fix — experimental v0.1

Bridge Android específico para o EasySMX X05 quando ele se anuncia via Bluetooth como `045e:02e0` mas a Web Gamepad API/xCloud recebe um layout incorreto.

## O que faz
1. UserService do Shizuku roda como UID shell.
2. Procura automaticamente `/dev/input/event*` por Bus Bluetooth, VID `045e`, PID `02e0`.
3. Faz `EVIOCGRAB` do controle original para impedir entrada dupla.
4. Cria via `/dev/uinput` um `Microsoft X-Box 360 pad` virtual (`045e:028e`).
5. Repassa ABXY, LB/RB, Back/Start, L3/R3, sticks, LT/RT e D-pad.

## Build pelo GitHub Actions
Crie um repositório, envie todo este diretório e abra **Actions > Build APK > Run workflow**. O APK aparecerá em **Artifacts > X05-xCloud-Fix-debug**.

## Teste
- Inicie o Shizuku por Depuração sem fio.
- Pare/desinstale outros remapeadores.
- Conecte o X05 em Bluetooth/XInput.
- Abra X05 xCloud Fix, conceda Shizuku e toque **ATIVAR CORREÇÃO**.
- Se aparecer `OK: EVIOCGRAB ativo` e `Xbox 360 pad virtual criado`, abra HardwareTester.
- O controle virtual deve ser `Microsoft X-Box 360 pad`, `mapping: standard`.
- Esperado: A B0, B B1, X B2, Y B3, LB B4, RB B5, LT B6, RT B7, Back B8, Start B9, L3 B10, R3 B11, D-pad B12–B15.

## Segurança / reversão
Nenhum arquivo de `/system` é alterado. Ao tocar DESATIVAR ou encerrar o serviço, o uinput virtual é destruído e o `EVIOCGRAB` é liberado. Se o processo for morto de forma abrupta, o kernel fecha os file descriptors e libera o grab.

## Limitações v0.1
- Feito especificamente para o perfil Bluetooth `045e:02e0` observado no X05 testado.
- Sem rumble no virtual nesta primeira versão.
- A conversão dos eixos assume sticks 0..65535 e triggers 0..1023, conforme observado no dispositivo diagnosticado. Se uma revisão de firmware usar ranges diferentes, a escala precisa ser adaptada.
- O comportamento depende da política SELinux do aparelho permitir EVIOCGRAB e `/dev/uinput` ao UID shell.
