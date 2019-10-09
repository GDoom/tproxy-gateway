FROM multiarch/alpine:aarch64-edge 

ENV TZ=Asia/Shanghai

RUN	sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories && \
	apk --no-cache --no-progress upgrade && \
	apk --no-cache --no-progress add perl curl bash gawk iptables ip6tables pcre openssl dnsmasq ipset iproute2 tzdata && \
	sed -i 's/mirrors.aliyun.com/dl-cdn.alpinelinux.org/g' /etc/apk/repositories && \
	ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

RUN mkdir -p /v2ray && \
	cd /v2ray && \
	tag_url="https://api.github.com/repos/v2ray/v2ray-core/releases/latest" && \
	new_ver=`curl -s ${tag_url} --connect-timeout 10| grep 'tag_name' | cut -d\" -f4` && \
	new_ver="v${new_ver##*v}" && \
	wget https://github.com/v2ray/v2ray-core/releases/download/${new_ver}/v2ray-linux-arm64.zip && \
	unzip v2ray-linux-arm64.zip && \
	rm config.json v2ray-linux-arm64.zip && \
	chmod +x v2ray v2ctl && mkdir -p /sample_config

RUN cd / && mkdir -p /ss-tproxy &&\
	#release_url=$(curl -Ls -o /dev/null -w %{url_effective} https://github.com/zfl9/ss-tproxy/releases/latest) && \
	#tag="${release_url##*/}" && \
	#wget -O ss-tproxy.zip "https://github.com/zfl9/ss-tproxy/archive/$tag.zip" && \
	wget -O ss-tproxy.zip https://github.com/zfl9/ss-tproxy/archive/master.zip && \
	unzip -jd ss-tproxy ss-tproxy.zip && rm ss-tproxy.zip && \
	sed -i 's/while umount \/etc\/resolv.conf \&>\/dev\/null; do true; done/while mount|grep overlay|grep \/etc\/resolv.conf \&>\/dev\/null; do umount \/etc\/resolv.conf \&>\/dev\/null; done/g' /ss-tproxy/ss-tproxy /ss-tproxy/ss-tproxy && \
	sed -i 's/60053/53/g' /ss-tproxy/ss-tproxy && \
	sed -i '/no-resolv/i\addn-hosts=$dnsmasq_addn_hosts' /ss-tproxy/ss-tproxy && \
	install -c /ss-tproxy/ss-tproxy /usr/local/bin && \
	mkdir -m 0755 -p /etc/ss-tproxy && chown -R root:root /etc/ss-tproxy && \
	install -c /ss-tproxy/ss-tproxy.conf /ss-tproxy/gfwlist.* /ss-tproxy/chnroute.* /etc/ss-tproxy && \
	rm -rf /ss-tproxy

COPY koolproxy/arm /koolproxy
#RUN mkdir -p /koolproxy && cd /koolproxy && \
	#wget https://koolproxy.com/downloads/arm && \
    #mv arm koolproxy && \
RUN cd /koolproxy && \
	chmod +x koolproxy && \
	chown -R daemon:daemon /koolproxy

COPY init.sh /
COPY chinadns.aarch64 /tmp/chinadns
COPY ss-tproxy.conf v2ray.conf gfwlist.ext /sample_config/

RUN chmod +x /init.sh && \
	install -c /tmp/chinadns /usr/local/bin && \
	rm -rf /tmp/*

CMD ["/init.sh","daemon"]