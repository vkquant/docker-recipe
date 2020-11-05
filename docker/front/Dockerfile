FROM node:15

ENV NODE_ENV production
WORKDIR /var/www/front

RUN apt-get update && apt-get install -y \
    git \
    ssh

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

COPY package.json /var/www/front
COPY yarn.lock /var/www/front
RUN mkdir -p /root/.ssh
COPY docker/front/keys/id_rsa.pub /root/.ssh/id_rsa.pub
COPY docker/front/keys/id_rsa /root/.ssh/id_rsa
RUN chmod 600 /root/.ssh/id_rsa.pub
RUN chmod 600 /root/.ssh/id_rsa
RUN ssh-keyscan bitbucket.org > /root/.ssh/known_hosts
RUN ssh -T git@bitbucket.org
RUN ssh -v git@bitbucket.org

COPY . /var/www/front

ENV PATH /var/www/front/node_modules/.bin:$PATH

EXPOSE 3000

### Only for build images ###

#### Build application ####
RUN yarn install
RUN yarn add webpack-dev-server -g

CMD yarn start
