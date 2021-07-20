FROM octoprint/octoprint:latest

ENV PIP_USER false

RUN apt-get update && apt-get install -y \
    libpq-dev

# Install plugins
RUN pip install \
    "https://github.com/eyal0/OctoPrint-PrintTimeGenius/archive/master.zip" \
    "https://github.com/OllisGit/OctoPrint-DisplayLayerProgress/releases/latest/download/master.zip" \
    "https://github.com/j7126/OctoPrint-Dashboard/archive/master.zip" \
    "https://github.com/BillyBlaze/OctoPrint-TouchUI/archive/master.zip"\
    "https://github.com/birkbjo/OctoPrint-Themeify/archive/master.zip"\
    psycopg2
