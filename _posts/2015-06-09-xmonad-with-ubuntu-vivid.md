---
title: "Ubuntu VividでXMonadとGnomeを連携させる設定"
subtitle: "タイル型ウィンドウマネージャーで聖徳太子を目指す話"
date: 2015-06-09 08:25:00
---
UbuntuでXMonadを使い始めてからかれこれ1年ぐらい経ったと思う。[XMonad](//xmonad.org)はタイル型ウィンドウマネージャの一種であり、広く普及してるスタック型ウィンドウマネージャーと異なりウィンドウが重ならいように自動的に配置され、画面スペースをより有効に使用することが可能になる。スタック型ウィンドウマネージャーでは通常ウィンドウを3〜4つ開いてると重なってしまい、いちいち切り替えが多くなってしまう。タイル型ウィンドウマネージャーならウェブをブラウジングしつつTwitterを確認し、ついでに動画を見ながら実験レポートに手をつけるのも朝飯前だ。まあもっともその状態でレポートで進捗を出すためには聖徳太子並のスペックが必要になるかもしれないが。

![]({{ site.baseurl }}/img/2015-06/xmonad.png "聖徳太子仕様")

XMonadはウィンドウに関する操作を簡単なキーボード操作で済ませてくれる。しかしXMonadを快適に使用できる前にいくつか設定をする必要がある。特にUbuntuをアップグレードするとほぼ必ず何かが動かなくなり設定を変更する必要が出てくる。一応このブログもメモと称してるので今後忘れたときの為に、加えた変更についてメモを残したい。この記事ではUbuntu 15.04にアップグレードした時にやるべきことを書いておく。

Ubuntu 14.10まではxmonadのパッケージをインストールするとXMonadがGnome-Flashback-Sessionで使えるように設定された。しかし15.04にアップグレードしたらGnomeと連携する部分がごっそり消えてることに気づく。一応調べてみたらGnome3になってからxmonadとGnomeを連携させるのに苦労して[放棄された](//bazaar.launchpad.net/~ubuntu-branches/ubuntu/vivid/xmonad/vivid/revision/28)様子だった。ただ15.04にアップグレードした人もこの記事にしたがって設定をすればXMonadとGnomeを連携させることができる。ただGnome3とかUnityとかちょくちょく仕様が変わるから次15.10とかにアップグレードしたら十中八九設定を変更しないとまたうまく動かなくなるだろう。

# 手順
ここから本題に入る。まずは15.04で消されたファイルをいくつか作りなおす必要がある。最初にXセッションファイルを作る。 `/usr/share/xsessions/gnome-xmonad.desktop` に以下の内容を保存する。

	[Desktop Entry]
	Name=GNOME Xmonad
	Comment=This session logs you into GNOME Flashback with xmonad
	Exec=gnome-session --session=gnome-xmonad
	TryExec=gnome-session
	Icon=
	Type=Application
	DesktopNames=GNOME-Flashback;Unity;
	X-Ubuntu-Gettext-Domain=gnome-xmonad

本当は `/usr/share` はaptの管轄なので `/usr/local/share/xsessions/gnome-xmonad.desktop` とかに保存したかったがデフォルトの設定ではそこからセッションファイルは読み込まれないみたいだったので断念した。

次にGnomeセッションファイルを作る。 `/usr/local/share/gnome-session/sessions/gnome-xmonad.session` に以下の内容を保存する。

	[GNOME Session]
	Name=Xmonad/GNOME
	RequiredComponents=gnome-flashback-init;gnome-flashback;gnome-panel;xmonad;unity-settings-daemon;gnome-flashback-services;
	DesktopName=Unity

これも15.04で消されたファイルの一つである。もっとも14.10ではこのファイルは存在してたが、 `gnome-setting-daemon` を `unity-settings-daemon` に変更したり `DesktopName=Unity` を追加しないと正しく動かなかったため結局 `/usr/local` 以下に似たようなファイルを置く必要があった記憶がある。

以上の手順を踏んでもデスクトップが出るまで異常に時間がかかったりうまく動かなかったりした場合には `~/.xmonad/lib/XMonad/Config/Gnome.hs` に以下の内容を保存しよう:
```haskell
{-# OPTIONS_GHC -fno-warn-missing-signatures #-}

-----------------------------------------------------------------------------
-- |
-- Module       : XMonad.Config.Gnome
-- Copyright    : (c) Spencer Janssen <spencerjanssen@gmail.com>
-- License      : BSD
--
-- Maintainer   : Spencer Janssen <spencerjanssen@gmail.com>
-- Stability    :  unstable
-- Portability  :  unportable
--
-- This module provides a config suitable for use with the GNOME desktop
-- environment.

module XMonad.Config.Gnome (
    -- * Usage
    -- $usage
    gnomeConfig,
    gnomeRun,
    gnomeRegister
    ) where

import XMonad
import XMonad.Config.Desktop
import XMonad.Util.Run (safeSpawn)

import qualified Data.Map as M

import System.Environment (getEnvironment)

-- $usage
-- To use this module, start with the following @~\/.xmonad\/xmonad.hs@:
--
-- > import XMonad
-- > import XMonad.Config.Gnome
-- >
-- > main = xmonad gnomeConfig
--
-- For examples of how to further customize @gnomeConfig@ see "XMonad.Config.Desktop".

gnomeConfig = desktopConfig
    { terminal = "gnome-terminal"
    , keys     = gnomeKeys <+> keys desktopConfig
    , startupHook = gnomeRegister >> startupHook desktopConfig }

gnomeKeys (XConfig {modMask = modm}) = M.fromList $
    [ ((modm, xK_p), gnomeRun)
    , ((modm .|. shiftMask, xK_q), spawn "gnome-session-save --kill") ]

-- | Launch the "Run Application" dialog.  gnome-panel must be running for this
-- to work.
gnomeRun :: X ()
gnomeRun = withDisplay $ \dpy -> do
    rw <- asks theRoot
    gnome_panel <- getAtom "_GNOME_PANEL_ACTION"
    panel_run   <- getAtom "_GNOME_PANEL_ACTION_RUN_DIALOG"

    io $ allocaXEvent $ \e -> do
        setEventType e clientMessage
        setClientMessageEvent e rw gnome_panel 32 panel_run 0
        sendEvent dpy rw False structureNotifyMask e
        sync dpy False

-- | Register xmonad with gnome. 'dbus-send' must be in the $PATH with which
-- xmonad is started.
--
-- This action reduces a delay on startup only only if you have configured
-- gnome-session>=2.26: to start xmonad with a command as such:
--
-- > gconftool-2 -s /desktop/gnome/session/required_components/windowmanager xmonad --type string
gnomeRegister :: MonadIO m => m ()
gnomeRegister = io $ do
    x <- lookup "DESKTOP_AUTOSTART_ID" `fmap` getEnvironment
    whenJust x $ \sessionId -> safeSpawn "dbus-send"
            ["--session"
            ,"--print-reply=literal"
            ,"--dest=org.gnome.SessionManager"
            ,"/org/gnome/SessionManager"
            ,"org.gnome.SessionManager.RegisterClient"
            ,"string:xmonad"
            ,"string:"++sessionId]
```
これは[XMonadとGnomeを連携させるためのモジュール](//xmonad.org/xmonad-docs/xmonad-contrib/XMonad-Config-Gnome.html)だが、残念ながらxmonadのパッケージにあらかじめあるコードでは動かない。どうやら `dbus-send` の仕様が変わり、77行目が本来 `--print-reply=literal` にするべきなのが `--print-reply=string` になってるのが原因なのでオリジナルのソースからそこだけ変更してある。この修正が必要な場合は、確か `~/.xsession-errors` にdbus関連のエラーが出現するはずだから一応確認してみるのもいいかもしれない。どうやら去年の段階で[パッチが送られてる様子](//mail.haskell.org/pipermail/xmonad/2014-May/014130.html)なのでこのファイルを自分で用意する必要はそろそろなくなるだろう。

# Super+Pキーコンビネーションについて
XMonadのmodキーをSuperキー(Windowsキー)に指定してる人は、Super+pを押しても設定どおりに動かず画面表示がおかしくなることに気づくだろう。これはマイクロソフトがモニター切り替えボタンでSuper+pのキーコンビネーションを送信するよう[ハードウェアメーカーに推奨](//mjg59.livejournal.com/121851.html)してるからである。これを受け、多くのgnome-setings-daemonはSuper+pのキーコンビネーションを検知すると画面出力を再構成してしまう。これを無効化するためには `dconf write /org/gnome/settings-daemon/plugins/xrandr/active false` を実行すればいい。実際に画面表示の切替をする必要がある人は無効化などせずXMonadの設定を変更し別のキーにバインドした方がいいだろう。自分はディスプレイ1つしか使用しないため、無効化してる。

# 最後に
XMonadのインストール方法についてついて知りたい人は[ここ](//wiki.haskell.org/Xmonad/Installing_xmonad)を見るといいだろう。XMonadとGnomeを連携させたい人は最低限次の設定を `~/.xmonad/xmonad.hs` に保存すればいい。 

```haskell
import XMonad
import XMonad.Config.Gnome

main = do
    xmonad $ gnomeConfig
```

XMonadの設定についてはネットに多くの資料が存在するため、より細かい設定をしたい人は調べて欲しい。参考までに自分の設定を以下に残しておく:

```haskell
import XMonad
import XMonad.Config.Desktop (desktopLayoutModifiers)
import XMonad.Config.Gnome
import XMonad.Hooks.DynamicLog
import XMonad.Hooks.ManageHelpers
import XMonad.Hooks.Minimize
import XMonad.Layout.BorderResize
import XMonad.Layout.ImageButtonDecoration
import XMonad.Layout.DecorationAddons
import XMonad.Layout.Minimize
import XMonad.Layout.Maximize
import XMonad.Layout.NoBorders
import XMonad.Layout.Named
import XMonad.Layout.SimpleDecoration
import XMonad.Layout.SimplestFloat
import XMonad.Util.EZConfig (additionalKeysP)

myFloat = named "Float" $ floatingDeco $ borderResize $ minimize $ maximize $ noBorders simplestFloat
	where floatingDeco = imageButtonDeco shrinkText defaultThemeWithImageButtons
		{ activeColor = "#000000"
		, inactiveColor = "#3C3B37"
		, fontName = "xft:TakaoPGothic-9:bold" }

myLayout = tiled ||| Mirror tiled ||| Full ||| myFloat 
  where
     -- default tiling algorithm partitions the screen into two panes
     tiled   = Tall nmaster delta ratio
     -- The default number of windows in the master pane
     nmaster = 1
     -- Default proportion of screen occupied by master pane
     ratio   = 1/2
     -- Percent of screen to increment by when resizing panes
     delta   = 3/100

myManageHook = composeAll
	[ manageHook gnomeConfig
	, isDialog --> doCenterFloat
	, isFullscreen --> doFullFloat
	]

myHandleEventHook = minimizeEventHook

main = do
	xmonad $ gnomeConfig
		{ manageHook = myManageHook
		, layoutHook = desktopLayoutModifiers $ smartBorders $ myLayout
		, handleEventHook = myHandleEventHook <+> handleEventHook gnomeConfig
		, workspaces = ["1", "2", "3", "4", "5", "6", "7", "8", "9"]
		, modMask = mod4Mask
		, focusFollowsMouse = False }
		`additionalKeysP`
		[ ("M-S-q", spawn "gnome-session-quit --power-off") ]
```

参考サイト:
[Gnome Xmonad broken after upgraded to 14.10 - ask ubuntu](//askubuntu.com/questions/541327/gnome-xmonad-broken-after-upgraded-to-14-10)
