ARG VERSION=latest
FROM ghcr.io/browserless/base:$VERSION
LABEL org.opencontainers.image.source=https://github.com/browserless/browserless

ENV PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1

COPY fonts/* /usr/share/fonts/truetype/
COPY src src/
RUN rm -r src/routes/
COPY src/routes/management src/routes/management/
COPY src/routes/chromium src/routes/chromium/

RUN echo "ttf-mscorefonts-installer msttcorefonts/accepted-mscorefonts-eula select true" | debconf-set-selections && \
  apt-get -y -qq install software-properties-common &&\
  apt-add-repository "deb https://archive.canonical.com/ubuntu $(lsb_release -sc) partner" && \
  apt-get -y -qq --no-install-recommends install \
  fontconfig \
  fonts-freefont-ttf \
  fonts-gfs-neohellenic \
  fonts-indic \
  fonts-ipafont-gothic \
  fonts-kacst \
  fonts-liberation \
  fonts-noto-cjk \
  fonts-noto-color-emoji \
  fonts-roboto \
  fonts-thai-tlwg \
  fonts-ubuntu \
  fonts-wqy-zenhei \
  fonts-open-sans

RUN EXPECTED_PATH=$(node -e "const playwright = require('playwright-core'); console.log(playwright.chromium.executablePath());") && \
  EXPECTED_DIR=$(dirname "${EXPECTED_PATH}") && \
  mkdir -p "${EXPECTED_DIR}" && \
  wget -O /tmp/chromium.tar.xz "https://github.com/adryfish/fingerprint-chromium/releases/download/139.0.7258.154/ungoogled-chromium-139.0.7258.154-1-x86_64_linux.tar.xz" && \
  tar -xJf /tmp/chromium.tar.xz -C "${EXPECTED_DIR}" --strip-components=1 && \
  chmod +x "${EXPECTED_PATH}" && \
  rm /tmp/chromium.tar.xz

# NOTE it's important to not use npx playwright-core here since it'll likely install
# a more recent version than we potentially have in our own package.json
RUN ./node_modules/playwright-core/cli.js install-deps chromium &&\
  npm run build &&\
  npm run build:function &&\
  npm prune production &&\
  npm run install:debugger &&\
  chown -R blessuser:blessuser $APP_DIR &&\
  fc-cache -f -v && \
  apt-get -qq clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/fonts/truetype/noto

USER blessuser

CMD ["./scripts/start.sh"]
