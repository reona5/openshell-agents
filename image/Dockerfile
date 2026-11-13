# OpenShell-Community の base に、版を固定した codex / claude と Codex 用の起動スクリプトを足したイメージ。
# base に入っている codex 0.117.0 は古く、ChatGPT アカウントで使えるモデルがない(2026-11 時点)。
ARG BASE_IMAGE=ghcr.io/nvidia/openshell-community/sandboxes/base:latest
FROM ${BASE_IMAGE}
USER root
ARG CODEX_VERSION=0.165.0
ARG CLAUDE_VERSION=2.1.318
RUN npm install -g @openai/codex@${CODEX_VERSION} && codex --version
RUN curl -fsSL https://claude.ai/install.sh | bash -s ${CLAUDE_VERSION} \
    && cp -L /root/.local/bin/claude /usr/local/bin/claude \
    && chmod 755 /usr/local/bin/claude && /usr/local/bin/claude --version
COPY codex-in-openshell /usr/local/bin/codex-in-openshell
RUN chmod 755 /usr/local/bin/codex-in-openshell
# 自動更新は止める。イメージの ENV は sandbox のプロセスに渡らないので、
# DISABLE_AUTOUPDATER=1 は osa new が sandbox の作成時に --env で渡す
USER sandbox
# 初回のテーマ選択と注意事項の画面を飛ばす。フォルダの信頼は osa new が clone 先にだけ付ける
RUN node -e 'const f=process.env.HOME+"/.claude.json";const fs=require("fs");const j=fs.existsSync(f)?JSON.parse(fs.readFileSync(f)):{};Object.assign(j,{hasCompletedOnboarding:true,theme:"dark"});fs.writeFileSync(f,JSON.stringify(j,null,2))'
