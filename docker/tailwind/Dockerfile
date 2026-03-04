FROM oven/bun:alpine

WORKDIR /app
RUN bun add @tailwindcss/cli

ENV NODE_PATH=/app/node_modules

ENTRYPOINT ["/app/node_modules/.bin/tailwindcss"]
