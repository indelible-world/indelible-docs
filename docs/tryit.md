---
title: Try It Out
icon: lucide/flask-conical
---

# Try It Out

## Indelible App

With an [Ethereum-compatible][wallet] [browser wallet](https://ethereum.org/wallets/find-wallet/) like [MetaMask](https://metamask.io) and some [Sepolia ETH](https://sepolia-faucet.pk910.de/), go certify some text on our open-source webapp.

Then, verify that same text using our verification tool, no browser wallet needed. You can also try it out using [this journal article.](assets/covidinfodemic.indelible.pdf)

[Indelible App for Verifying](https://app.indelible.world/?text=hello world!){ .md-button .md-button--primary} [Indelible App for Certifying](https://app.indelible.world/publish){ .md-button}

## Indelible Demo Page

See what Indelible looks like in the real world with our demo site and browser extension:

[Demo Site](https://demo.indelible.world/){ .md-button .md-button--primary} [Browser Extension (Chrome)](https://chromewebstore.google.com/detail/indelible-verifier/ejifedijopaffofkjohhnkebbfgeefcp){ .md-button}  [Browser Extension (Firefox)](https://addons.mozilla.org/en-US/firefox/addon/indelible-verifier/){ .md-button}

??? info "Install the beta version of our browser extension"
    Want to get early access to some upcoming verification features? Follow these steps to install our nightly build.
    === "Chrome"
        1. Download the [linked .zip file](https://github.com/indelible-world/indelible-browser-extension/releases/download/latest/indelible-browser-extension-chrome-latest.zip) above
        1. Unzip the folder
        1. Go to [chrome://extensions/](chrome://extensions/) (copy this link and paste it into your browser bar - Chrome doesn't allow linking directly to internal pages)
        1. Enable Developer Mode if it is not already on
        1. Click "Load unpacked"
        1. Select the folder you just unzipped
        1. Click the extension puzzle piece to the right of your address bar and click the pin button next to Indelible Verifier

    === "Firefox"
        1. Download the [linked .zip file](https://github.com/indelible-world/indelible-browser-extension/releases/download/latest/indelible-browser-extension-firefox-latest.zip) above
        1. Go to [about:debugging#/runtime/this-firefox](about:debugging#/runtime/this-firefox) (copy this link and paste it into your browser bar - Firefox doesn't allow linking directly to internal pages)
        1. Click "Load Temporary Add-on"
        1. Select the .zip file you just downloaded


[wallet]: https://eips.ethereum.org/EIPS/eip-1193 "We support all EIP-1193-compatible browser wallets."