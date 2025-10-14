FROM jenkins/jenkins:lts

# Install plugins automatically from plugins.txt
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt
RUN jenkins-plugin-cli --plugin-file /usr/share/jenkins/ref/plugins.txt

# Copy JCasC configuration
COPY casc.yaml /usr/share/jenkins/ref/casc.yaml

ENV CASC_JENKINS_CONFIG=/usr/share/jenkins/ref/casc.yaml

# Optionally, set JAVA_OPTS for disabling setup wizard
ENV JAVA_OPTS="-Djenkins.install.runSetupWizard=false"

# Best practice: install required CLI tools directly (Node.js LTS and SonarScanner CLI)
USER root
RUN apt-get update \
	&& apt-get install -y --no-install-recommends curl wget unzip ca-certificates xz-utils \
	&& rm -rf /var/lib/apt/lists/*

# Install Node.js 18 LTS
ENV NODE_VERSION=18.20.2
RUN ARCH=$(dpkg --print-architecture) \
	&& case "$ARCH" in \
		 amd64) NODE_ARCH=x64 ;; \
		 arm64) NODE_ARCH=arm64 ;; \
		 *) echo "Unsupported arch: $ARCH" && exit 1 ;; \
	   esac \
	&& curl -fsSL https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-${NODE_ARCH}.tar.xz -o /tmp/node.tar.xz \
	&& mkdir -p /usr/local/lib/nodejs \
	&& tar -xJf /tmp/node.tar.xz -C /usr/local/lib/nodejs \
	&& ln -s /usr/local/lib/nodejs/node-v${NODE_VERSION}-linux-${NODE_ARCH}/bin/node /usr/local/bin/node \
	&& ln -s /usr/local/lib/nodejs/node-v${NODE_VERSION}-linux-${NODE_ARCH}/bin/npm /usr/local/bin/npm \
	&& ln -s /usr/local/lib/nodejs/node-v${NODE_VERSION}-linux-${NODE_ARCH}/bin/npx /usr/local/bin/npx \
	&& rm -f /tmp/node.tar.xz

# Install SonarScanner CLI
ENV SONAR_SCANNER_VERSION=5.0.1.3006
RUN SCAN_URL="https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-${SONAR_SCANNER_VERSION}-linux.zip" \
	&& curl -fsSL "$SCAN_URL" -o /tmp/sonar-scanner.zip \
	&& unzip /tmp/sonar-scanner.zip -d /opt \
	&& ln -s /opt/sonar-scanner-${SONAR_SCANNER_VERSION}-linux/bin/sonar-scanner /usr/local/bin/sonar-scanner \
	&& rm -f /tmp/sonar-scanner.zip

# Return to jenkins user
USER jenkins

# Expose default Jenkins ports
EXPOSE 8080 50000
