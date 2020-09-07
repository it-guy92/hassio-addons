ARG REGISTRY_URL
ARG BUILD_IMAGE
ARG BUILD_TAG
ARG BUILD_FROM

FROM ${REGISTRY_URL}/${BUILD_IMAGE}:${BUILD_TAG} as build
ARG BUILD_ARCH
ARG BUILD_VERSION
ARG GIT_REPO_URL
RUN git clone --recursive --depth 1 -q ${GIT_REPO_URL} /hyperion || { echo "---> Failed to download Hyperion source code! Abort"; exit 1; }
RUN /bin/bash -c "cd /hyperion && mkdir build && cd build && \
	cmake -DCMAKE_BUILD_TYPE=Release \
		  -DENABLE_QT=OFF -DENABLE_XCB=OFF -DENABLE_X11=OFF -DENABLE_CEC=OFF -DENABLE_V4L2=OFF -DENABLE_FB=OFF \
		  .. || exit 2 && \
	make -j $(nproc) || exit 3 && \
    strip /hyperion/build/bin/h* && \
	exit 0; \
	exit 1 " || { echo "---> Hyperion compilation failed! Abort"; exit 4; }


FROM ${BUILD_FROM} as run
ENV LANG C.UTF-8
RUN apt-get update && apt-get install -y --no-install-recommends \
		libpython3.7 \
		libqt5widgets5 \
		libqt5serialport5 \
		libqt5sql5-sqlite \
		libqt5x11extras5 \
		libavahi-core7 \
		libavahi-compat-libdnssd1 \
		libusb-1.0-0 \
		libturbojpeg0 \
		libssl1.1 \
		zlib1g \
    && rm -rf /var/lib/apt/lists/*
COPY --from=build /hyperion/build/bin/h* /hyperion/
COPY run.sh /
RUN chmod a+x /run.sh
WORKDIR /hyperion
CMD [ "/run.sh" ]