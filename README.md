repo: https://github.com/end-4/dots-hyprland

O que é o repositório

dots-hyprland (também chamado "illogical-impulse") são os dotfiles do usuário end-4 para Hyprland (compositor Wayland), com foco numa "shell" gráfica completa construída em Quickshell — não é um instalador de sistema (drivers, partições etc.), apenas configurações + um script que copia tudo para o lugar certo.

Estrutura de pastas (raiz)
Pasta/arquivo Função
dots/ O conteúdo real que é copiado para o seu $HOME (replica ~/.config e ~/.local)
dots-extra/ Configs opcionais extras (emacs, fcitx5, fedora, swaylock, nix) — não instaladas por padrão
sdata/ Scripts e dados do instalador (./setup), listas de dependências por distro
licenses/ Licenças de trechos copiados de outros projetos
setup Script bash principal — ./setup install, uninstall, exp-update etc.
diagnose Script para gerar relatório de diagnóstico (ajuda em issues)
dots/.config/ — o que você vai editar

1. hypr/ — configuração do Hyprland em Lua (não .conf puro; usa hyprlang+Lua via um pré-processador):

hyprland.lua → arquivo raiz que importa os outros
hyprland/{env,general,keybinds,rules,variables,execs}.lua → configs "de fábrica" (não mexer, são sobrescritos em updates)
custom/{env,general,keybinds,rules,variables,execs}.lua → é aqui que você edita suas customizações (atalhos, regras de janela, autostart), sem perder ao atualizar
hyprlock.conf / hypridle.conf → tela de bloqueio e idle

2. quickshell/ii/ — o "shell" gráfico em si (QML), responsável pela barra, dock, overview, sidebars, notificações, IA, etc:

shell.qml → ponto de entrada
modules/ii/ → cada widget/painel tem sua própria pasta: bar/, dock/, overview/, sidebarLeft/, sidebarRight/, notificationPopup/, onScreenDisplay/, lock/, cheatsheet/, wallpaperSelector/, etc.
modules/common/ → coisas compartilhadas: Appearance.qml (cores/estilo), Config.qml (lê o JSON de config do usuário), Directories.qml, widgets/ (componentes reutilizáveis tipo botões, sliders)
modules/settings/ → as telas do app de configurações gráfico (o "Settings app" mostrado nos screenshots)
services/ → lógica de backend em QML: Audio, Battery, Network, Notifications, Ai.qml, Weather.qml, Mpris (players de mídia), Cliphist (clipboard), etc. — é aqui que ficam as integrações com o sistema
defaults/ai/ → prompts padrão usados pela IA integrada

3. matugen/ — gera o esquema de cores Material You a partir do seu wallpaper e aplica em GTK, Hyprland, Kvantum, fuzzel, kde etc. (templates em matugen/templates/)

4. Outras configs de app: kitty/, foot/ (terminais), fuzzel/ (launcher), fish/, zshrc.d/, starship.toml (prompt), wlogout/, mpv/, Kvantum/ + kdeglobals (tema Qt)

Como editar sem quebrar updates
Para Hyprland: edite só os arquivos em hypr/custom/_.lua — os de hypr/hyprland/_.lua são sobrescritos em ./setup exp-update.
Para o shell (aparência, comportamento dos widgets): existe um app de Configurações gráfico (modules/settings/) que grava num JSON de config lido por Config.qml — mudanças ali sobrevivem a updates. Editar diretamente os .qml dos widgets funciona, mas pode ser sobrescrito num update (a menos que você use ./setup exp-merge, que tenta fazer rebase preservando suas mudanças locais).

Executou um comando

Boa visão geral. Agora vou olhar cada painel com mais profundidade (como é montado no shell, onde entra a config do usuário) e depois toda a parte de suspensão/hibernação.
Executou 6 comandos · 1 nota

Agora vou checar Idle.qml (serviço) e SessionWarnings, e ver se há algo no sdata/subcmd-install relacionado a systemd-logind/sleep hooks (como restauração de wallpaper após suspender, que apareceu no script __restore_video_wallpaper.sh).
Executou 6 comandos · 1 nota
Arquitetura de painéis

Tudo entra pelo shell.qml, que carrega uma "panel family" — hoje existem duas: ii (padrão, definida em panelFamilies/IllogicalImpulseFamily.qml) e waffle (alternativa, estilo diferente). Você pode alternar com Super (atalho panelFamilyCycle).

Dentro de IllogicalImpulseFamily.qml, cada painel é instanciado via um PanelLoader — um wrapper que só carrega o componente se Config.options.<algo> permitir (ex: Bar só carrega se !Config.options.bar.vertical, Dock só se Config.options.dock.enable). Isso significa que, para ligar/desligar um painel inteiro, normalmente basta mudar uma opção no app de Configurações — não precisa editar QML.
Cada widget/painel (modules/ii/)
Pasta O que é Arquivo de entrada
bar/ Barra superior horizontal: relógio, workspaces, tray, indicador de bateria, recursos (CPU/RAM), clima, mídia Bar.qml → BarContent.qml → BarGroup.qml (agrupa "pílulas")
verticalBar/ Versão vertical da barra (alternativa, ativada por Config.options.bar.vertical) VerticalBar.qml
background/ Camada de fundo/wallpaper — widgets soltos que podem ficar sobre o wallpaper (relógio grande, etc.) Background.qml
dock/ Dock estilo macOS com apps fixados/abertos Dock.qml → DockApps.qml
overview/ Visão geral tipo "Mission Control": mostra todas as janelas/workspaces com preview ao vivo + busca de apps Overview.qml, SearchBar.qml
sidebarLeft/ Sidebar com chat de IA (AiChat.qml), tradutor (Translator.qml), e um recurso de anime/imagens (booru) SidebarLeft.qml
sidebarRight/ Sidebar de "central de controle": toggles rápidos (wifi/bluetooth/DND), calendário, pomodoro, todo list, mixer de volume, notificações SidebarRight.qml
notificationPopup/ Popups de notificação (toast) que aparecem no canto NotificationPopup.qml
onScreenDisplay/ OSD de volume/brilho/gama que aparece ao usar teclas de mídia OnScreenDisplay.qml + indicators/
onScreenKeyboard/ Teclado virtual na tela (útil em touch) OnScreenKeyboard.qml
mediaControls/ Widget flutuante de controle de mídia (MPRIS) MediaControls.qml
cheatsheet/ Tela de "colinha" com todos os atalhos de teclado (lida os binds do Hyprland) + bônus: tabela periódica Cheatsheet.qml, CheatsheetKeybinds.qml
wallpaperSelector/ Seletor visual de papel de parede (dispara o Matugen para gerar cores) WallpaperSelector.qml
lock/ Tela de bloqueio nativa do Quickshell (alternativa ao hyprlock) Lock.qml, LockSurface.qml
sessionScreen/ Menu de energia: lock/sleep/logout/hibernate/shutdown/reboot (ver seção abaixo) SessionScreen.qml
polkit/ Diálogo gráfico de autenticação (senha root) via polkit agent Polkit.qml
regionSelector/ Seleção de região de tela (para screenshot/gravação) RegionSelector.qml
screenTranslator/ OCR + tradução de texto capturado na tela ScreenTranslator.qml
screenCorners/ Desenha cantos arredondados "falsos" sobre a tela (efeito visual) ScreenCorners.qml
overlay/ Um "hub" de mini-ferramentas sobrepostas: crosshair de mira, imagem flutuante, limitador de FPS, bloco de notas, gravador de tela, mixer de volume Overlay.qml (carrega os módulos filhos por OverlayWidgetDelegateChooser.qml)

Todos compartilham peças de modules/common/:

    Appearance.qml — paleta de cores Material You, fontes, arredondamento, animações (o que você mexe para estilizar globalmente)
    Config.qml — lê/escreve o JSON de configurações do usuário (fonte de verdade das opções ligadas ao app de Configurações)
    widgets/ — componentes reutilizáveis (StyledText, botões, etc.) usados por todos os painéis acima

Suspensão / Hibernação / Bloqueio de tela

Isso não é controlado pelo Quickshell diretamente — é o hypridle (daemon de idle do ecossistema Hyprland) que dispara os comandos, e o Quickshell só reage a eles.

1. hypr/hypridle.conf — o timer central

$lock_cmd = hyprctl dispatch 'hl.dsp.global("quickshell:lock")' & pidof qs quickshell hyprlock || hyprlock
$suspend_cmd = systemctl suspend || loginctl suspend

general {
lock_cmd = $lock_cmd
before_sleep_cmd = loginctl lock-session
after_sleep_cmd = hyprctl dispatch 'hl.dsp.global("quickshell:lockFocus")'
inhibit_sleep = 3
}

listener { timeout = 300 on-timeout = loginctl lock-session } # 5 min → trava a tela
listener { timeout = 600 on-timeout = dpms disable / on-resume = dpms enable } # 10 min → desliga o monitor
listener { timeout = 900 on-timeout = $suspend_cmd } # 15 min → suspende (sleep)

Editar aqui é o lugar certo para mudar os tempos de bloqueio/suspensão — basta ajustar timeout = 300 etc. (valores em segundos). Note que não existe listener de hibernação por timeout — hibernação só é acionada manualmente.

    lock_cmd: tenta usar a tela de bloqueio do próprio Quickshell (quickshell:lock, o módulo lock/Lock.qml) e cai para hyprlock como fallback.
    inhibit_sleep = 3 é uma flag de bitmask do hypridle que diz para ignorar certos inibidores de idle (não relacionado a hibernação).

2. hypr/hyprlock.conf — visual da tela de bloqueio (fallback hyprlock)

Define o campo de senha, indicador de Caps Lock (hyprlock/check-capslock.sh), cores (hyprlock/colors.conf, gerado pelo Matugen a partir do wallpaper). 3. modules/ii/lock/ — tela de bloqueio nativa do shell

Lock.qml / LockSurface.qml é a alternativa em QML ao hyprlock, acionada via IPC (quickshell:lock). Se você preferir sempre usar o hyprlock tradicional, pode comentar essa parte no lock_cmd do hypridle.conf. 4. modules/common/functions/Session.qml — onde os comandos reais ficam

Este é o arquivo central para editar comportamento de energia. Cada ação é literalmente um comando de shell:
qml

function suspend() { Quickshell.execDetached(["bash","-c","systemctl suspend || loginctl suspend"]) }
function hibernate() { Quickshell.execDetached(["bash","-c","systemctl hibernate || loginctl hibernate"]) }
function lock() { Quickshell.execDetached(["loginctl","lock-session"]) }
function poweroff() { closeAllWindows(); Quickshell.execDetached([...poweroff...]) }
function reboot() { closeAllWindows(); Quickshell.execDetached([...reboot...]) }
function rebootToFirmware() { ...systemctl reboot --firmware-setup... }

Se quiser trocar systemctl por outro método (ex: zzz, s2ram, um script custom), é aqui que você edita. 5. modules/ii/sessionScreen/SessionScreen.qml — o menu visual

É o painel acionado por Ctrl+Alt+Delete (bind em keybinds.lua) que mostra os botões Lock / Sleep / Logout / Task Manager / Hibernate / Shutdown / Reboot / Reboot to firmware — cada botão só chama as funções do Session.qml acima. Editar textos/ícones/ordem dos botões é aqui. 6. Atalhos de teclado relacionados (hypr/hyprland/keybinds.lua, linhas ~334-341)
lua

hl.bind("CTRL + ALT + Delete", ...global("quickshell:sessionToggle")...) -- abre o menu
hl.bind("SUPER + L", exec_cmd("loginctl lock-session")) -- bloqueia direto
hl.bind("SUPER + SHIFT + L", exec_cmd("systemctl suspend || loginctl suspend")) -- sleep direto
-- hl.bind("switch:on:Lid Switch", exec_cmd("systemctl suspend..."), ...) -- COMENTADO: suspender ao fechar a tampa do notebook
hl.bind("CTRL+SHIFT+ALT+SUPER+Delete", exec_cmd("systemctl poweroff..."))

    Nota importante: o bind de "fechar a tampa" (Lid Switch) vem desativado por padrão (comentado), porque o comportamento padrão do systemd-logind já cuida disso na maioria das distros. Se sua distro não suspende ao fechar a tampa, é só descomentar essa linha em custom/keybinds.lua (nunca edite o arquivo hyprland/keybinds.lua original, que é sobrescrito em updates).

7. Inibição de suspensão (services/Idle.qml)

Um "singleton" com IdleInhibitor do protocolo Wayland — quando ativado (ex: assistindo vídeo em tela cheia, ou manualmente pelo usuário num toggle na sidebar direita), impede o hypridle de disparar bloqueio/DPMS/suspensão. O estado é persistido em Persistent.qml (sobrevive a reinícios do shell).
Resumo de "onde editar o quê"
Quero mudar... Edito
Tempo até travar / desligar monitor / suspender dots/.config/hypr/hypridle.conf
Comando usado para suspender/hibernar/desligar quickshell/ii/modules/common/functions/Session.qml
Aparência da tela de bloqueio (fallback) dots/.config/hypr/hyprlock.conf
Aparência do menu de sessão (botões) quickshell/ii/modules/ii/sessionScreen/SessionScreen.qml
Atalhos de lock/sleep/shutdown dots/.config/hypr/custom/keybinds.lua (não o de hyprland/)
Suspender ao fechar tampa do notebook descomentar linha do Lid Switch em custom/keybinds.lua

## Troubleshooting: Shell (Quickshell) não inicia após `yay -Syu`

### Sintomas

- O Hyprland inicia normalmente (workspaces, foco de janela funcionam).
- Os atalhos de teclado (`keybinds.lua`) funcionam.
- **A interface gráfica (barra, dock, sidebars, etc.) não aparece** — como se o Quickshell nunca tivesse subido.

### Causa

Isso geralmente acontece depois de um `yay -Syu` que atualiza o `qt6-base` e/ou `qt6-declarative` para uma build diferente da que o pacote `illogical-impulse-quickshell-git` foi compilado contra.

O Quickshell usa símbolos **privados** do Qt6 (`Qt_6_PRIVATE_API`), que não têm garantia de compatibilidade binária (ABI) entre atualizações — mesmo em patch-versions. Resultado: o binário `qs` falha ao carregar antes mesmo de tentar ler o `shell.qml`, então não sobra log de QML nenhum, só uma falha silenciosa no `exec-once` do Hyprland.

### Diagnóstico

1. Abra um terminal (ou troque de TTY com `Ctrl+Alt+F2`/`F3` se a sessão gráfica não tiver terminal disponível).
2. Rode o Quickshell manualmente para ver o erro real, em vez de deixar o Hyprland engolir a falha:
   ```bash
   qs -c ii
   ```
3. Se aparecer algo como:
   ```
   qs: symbol lookup error: qs: undefined symbol: _ZN23QUntypedPropertyBindingC1EP23QPropertyBindingPrivate, version Qt_6_PRIVATE_API
   ```
   confirma que é exatamente esse problema: **mismatch de ABI entre o Qt6 instalado e o binário do Quickshell**.

### Por que `yay -S illogical-impulse-quickshell-git --rebuild` não funciona

`illogical-impulse-quickshell-git` **não é um pacote do AUR**. É um PKGBUILD local que vive dentro do próprio repositório dos dotfiles, em:
```
sdata/dist-arch/illogical-impulse-quickshell-git/
```
O `(illogical-impulse)` que aparece no `yay -Qs` é um **grupo pacman** (`groups=(illogical-impulse)` no PKGBUILD), não o nome de um repositório — por isso `yay -S illogical-impulse-quickshell-git` retorna `No AUR package found`.

### Solução

Vá até a pasta desse PKGBUILD **dentro do clone local do dots-hyprland** (a mesma pasta usada quando você rodou `./setup install` pela primeira vez) e recompile diretamente com `makepkg`:

```bash
cd ~/dots-hyprland/sdata/dist-arch/illogical-impulse-quickshell-git
makepkg -Afsi --noconfirm
```

Flags:
- `-A` → ignora checagem de arquitetura
- `-f` → força rebuild mesmo se já existir um pacote buildado
- `-s` → resolve/instala dependências que faltarem via pacman
- `-i` → instala o pacote depois de compilar

Se não souber onde ficou o clone do repositório:
```bash
find ~ -maxdepth 3 -iname "dots-hyprland" -type d 2>/dev/null
```
Se não existir mais (foi apagado após a instalação), clone de novo:
```bash
git clone https://github.com/end-4/dots-hyprland.git
```

### Verificação

Depois do rebuild, teste de novo:
```bash
qs -c ii
```
Se o erro de símbolo sumir, mate qualquer instância antiga do Quickshell e deixe o Hyprland reexecutar o `exec-once` (ou reinicie a sessão gráfica):
```bash
pkill qs 2>/dev/null; pkill quickshell 2>/dev/null
```

### Prevenção

Depois de qualquer `yay -Syu` que mexa em pacotes `qt6-*`, vale recompilar o Quickshell **antes** de reiniciar a sessão gráfica:
```bash
cd ~/dots-hyprland/sdata/dist-arch/illogical-impulse-quickshell-git && makepkg -Afsi --noconfirm
```
Isso evita ficar sem interface gráfica depois de um update do sistema.
