FROM python:3 AS spo
WORKDIR /opt/
COPY spo-parser/ .
RUN apt-get update \
	&& apt-get install -y libgl1 ghostscript \
	&& pip install requests opencv-python camelot-py ghostscript
RUN ./run_extraction.sh



FROM alekzonder/puppeteer:latest AS pwaicons
USER root
WORKDIR /opt/
COPY rogue-thi-app/public/favicon.svg .
RUN mkdir ./splash && npx pwa-asset-generator --no-sandbox=true favicon.svg ./splash/



FROM node:14

WORKDIR /opt/next

ARG NEXT_PUBLIC_DISCORD_URLS
ARG NEXT_PUBLIC_MENSA_CHECKIN_LINK
ENV NEXT_PUBLIC_DISCORD_URLS $NEXT_PUBLIC_DISCORD_URLS
ENV NEXT_PUBLIC_MENSA_CHECKIN_LINK $NEXT_PUBLIC_MENSA_CHECKIN_LINK

COPY rogue-thi-app/package.json rogue-thi-app/package-lock.json ./
RUN npm install
COPY rogue-thi-app/ .
COPY --from=spo /opt/spo-grade-weights.json data/
COPY --from=pwaicons /opt/splash/ public/

RUN npm run build

USER node
EXPOSE 3000
CMD ["npm", "start"]