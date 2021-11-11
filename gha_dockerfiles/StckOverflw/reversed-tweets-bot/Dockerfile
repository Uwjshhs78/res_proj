FROM adoptopenjdk/openjdk15-openj9 as builder

COPY . .

RUN ./gradlew --no-daemon installDist

FROM adoptopenjdk/openjdk15-openj9

WORKDIR /user/app

COPY --from=builder build/install/reversetweets ./

ENTRYPOINT ["/user/app/bin/reversetweets"]