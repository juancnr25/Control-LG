name: Compilar APK

on:
  push:
    branches: [ main, master ]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Descomprimir proyecto
        run: |
          ZIP="$(ls *.zip | head -n 1)"
          unzip -o "$ZIP" -d fuente
          DIR="$(dirname "$(find fuente -maxdepth 3 -name settings.gradle | head -n 1)")"
          echo "SRC=$DIR" >> "$GITHUB_ENV"

      - uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Compilar
        working-directory: ${{ env.SRC }}
        run: gradle assembleDebug --no-daemon --stacktrace

      - name: Renombrar APK
        run: cp "${{ env.SRC }}/app/build/outputs/apk/debug/app-debug.apk" RemotoLG.apk

      - uses: actions/upload-artifact@v4
        with:
          name: RemotoLG
          path: RemotoLG.apk

      - name: Publicar en Releases
        uses: softprops/action-gh-release@v2
        with:
          tag_name: build-${{ github.run_number }}
          name: Remoto LG ${{ github.run_number }}
          files: RemotoLG.apk
          make_latest: true
