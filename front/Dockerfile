FROM node:6.6
MAINTAINER Samuel Loza "starsaminf@gmail.com"

# prepare a user which runs everything locally! - required in child images!
RUN useradd --user-group --create-home --shell /bin/false app

ENV HOME=/home/app
WORKDIR $HOME

RUN npm install -g npm && npm install -g @angular/cli  && npm cache clean

EXPOSE 4200
