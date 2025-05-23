# Lace E2E-tests

UI-mapped gherkin tests for the Lace browser extension

## Prerequisites

- Java 8+ (optional if you already have webdriver running on port 4444)
  - it is only required when using
    with [selenium-standalone](https://github.com/webdriverio/selenium-standalone/blob/main/docs/java-versions.md)
  - On macOS, you can install it easily with homebrew: `brew install openjdk`
- gpg
- Firefox Developer Edition (for running tests on Firefox locally)
- Docker (for Trezor tests)

## Running tests locally

- Set environment variable with wallet password that is used for test wallets and for decryption

  - `export WALLET_1_PASSWORD='<password>'`

- It is required to decrypt `walletConfiguration.ts.gpg` file that contains all the information about test wallets. (
  from the `packages/e2e-tests` directory)

  - `./decrypt_secret.sh`

- Install node dependencies (from the project root)

  - `yarn install`

- Build the extension for Chromium-based browsers (from the project root)

  - `yarn build`

- Build the extension for Firefox (from the project root)

  - `yarn build`
  - `cd apps/browser-extension-wallet`
  - `yarn build:firefox`

- Run tests (from the `packages/e2e-tests` directory)
  - `yarn test:local:chrome`
  - `yarn test:local:edge`
  - `yarn test:local:firefox`

## Selective gherkin scenario runs by tag matching

- `yarn wdio run wdio.conf.<browser>.ts --cucumberOpts.tags='@yourtag and @Testnet'`
- `yarn wdio run wdio.conf.<browser>.ts --cucumberOpts.tags='@yourtag or @otherTag and @Testnet'`
- `yarn wdio run wdio.conf.<browser>.ts --cucumberOpts.tags='@yourtag and not @Pending and @Testnet'`

## Supported browsers

- chrome
- edge
- firefox
  - does not support network interception or console log collection, so some tests/features are disabled (using `@skip(browserName="firefox")` tag)
  - works only with Firefox Developer Edition, as the regular version does not allow the use of extensions
  - does not support device emulation, so popup mode is simulated by simply resizing the window, which is not an ideal method of simulation

## Supported params

- `STANDALONE_DRIVER=true|false` default = false (optional)
  - true = use already running chromedriver/geckodriver on port 4444
  - false = use webdriver manager
- `ENV=(mainnet|preprod|preview)`default = preprod (optional)
  - determines default network used for tests
- `WALLET_1_PASSWORD=<password>`(required)
  - password for wallet, should match the password that is used in `walletConfiguration.ts` (can be set as an
    environment variable)
- `TEST_DAPP_URL=<url>`(required)
  - url for test DApp (only for DApp Connector tests)
- `SERVICE_WORKER_LOGS=true|false` default=false (optional)
  - enables service worker logs collection
  - not supported for Firefox
- `FIREFOX_BINARY=/Applications/Firefox\ Developer\ Edition.app/Contents/MacOS/firefox` (required for Firefox)
  - default path to Firefox Developer Edition binary on MacOS, please adjust to your local setup

## Run single feature file with params

- `ENV=preprod WALLET_1_PASSWORD='<password>' yarn wdio run wdio.conf.<browser>.ts --spec SendTransactionSimpleExtended.feature`

## Updating walletConfiguration.ts (for development)

- decrypt `walletConfiguration.ts.gpg` by running `./decrypt_secret.sh` (from the `packages/e2e-tests` directory)
- delete `packages/e2e-tests/src/support/walletConfiguration.ts.gpg`
- make necessary updates in `packages/e2e-tests/src/support/walletConfiguration.ts`
- encrypt `walletConfiguration.ts` by running `./encrypt_secret.sh` (from the `packages/e2e-tests` directory)
- delete `packages/e2e-tests/src/support/walletConfiguration.ts`

## Trezor test automation precondition (for local development)

- build docker image locally <https://github.com/input-output-hk/lace-hw-testing-toolkit.git> or
- `docker pull public.ecr.aws/e8d0p1a5/lw-hw-testing-toolkit:latest` (you need to be authenticated) and run it
- After starting docker image (info should be displayed - `⚡️ Trezor Device Manipulation API is running at http://localhost:8000`)
  Tests need to be triggered in 60 sec because emulator shuts down device if there is no action  

## Running tests locally in debug mode using IntelliJ IDEA/WebStorm

- create new run configuration
  - type: npm
- fill newly created configuration & environment variables as per attached screenshot
  - `example: STANDALONE_DRIVER=true;ENV=preprod;TEST_DAPP_URL=<yourUrl>;WALLET_1_PASSWORD=<walletPassword>`\
  ![debug1.png](src/images/readme/debug1.png)
- make sure you have a chromedriver running on port 4444 (in case of STANDALONE_DRIVER=true)
- add "@debug" tag to the cucumber test that you want to debug\
![debug3.png](src/images/readme/debug3.png)
- start debug run configuration you just created\
![debug2.png](src/images/readme/debug2.png)
