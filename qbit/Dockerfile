FROM lscr.io/linuxserver/qbittorrent
ARG CROSS_SEED_API_KEY
COPY --from=shared logging ./logging
COPY cross-seed ./cross-seed
COPY fertilizer ./fertilizer
RUN sed -i 's/YOUR_API_KEY/'"${CROSS_SEED_API_KEY}"'/g' ./cross-seed
RUN chmod a+x ./logging && chmod a+x ./cross-seed && chmod a+x ./fertilizer
