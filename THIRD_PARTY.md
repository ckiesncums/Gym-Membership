# Third-party software and fonts

- Bootstrap 5.3.8 — MIT, https://github.com/twbs/bootstrap/tree/v5.3.8. Local compiled stylesheet and license are in `public/assets/vendor/`.
- html5-qrcode 2.3.8 — Apache-2.0, https://github.com/mebjas/html5-qrcode/tree/v2.3.8. Local browser bundle and license are in `public/assets/vendor/`.
- chillerlan/php-qrcode 6.0.1 and php-settings-container 3.3.0 — MIT. Installed by Composer, locked in `composer.lock`; upstream licenses remain in `vendor/`.
- DM Sans and Barlow Condensed — SIL Open Font License 1.1. Local font files and original license notices are in `public/assets/fonts/`.

All application runtime assets are served locally. The scanner processes webcam/image pixels in the browser and sends only a QR token to this application's authenticated API.
