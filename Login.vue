<template>
    <div>
        <button @click="openLoginWindow" :disabled="isLoginDisabled">Login</button>
    </div>
</template>

<script>

    export default {
        name: "Login",
        data: () => {
            return {
                client_id: 'your_client_id', // Podio client id from the API section
                client_secret: 'your_client_secret', // Podio client secrets
                redirect_uri: `redirect_uri`, // URL which will open in the popup
                isLoginDisabled: false
            }
        },
        methods: {
            /**
             * Open login window
             * Handle redirects
             */
            openLoginWindow: function () {
                // disable login button from continuous click
                this.isLoginDisabled = true;
                //authentication end point
                let url = `https://podio.com/oauth/authorize?client_id=${this.client_id}&redirect_uri=${this.redirect_uri}`;
                // open login window
                let popup = window.open(url, '_blank', 'location=yes,height=570,width=520,scrollbars=yes,status=yes');
                // focus on the window
                if (popup && popup.focus) {
                    popup.focus();
                }
                // get the redirect uri path
                let redirectUriParser = document.createElement('a');
                redirectUriParser.href = this.redirect_uri;
                let redirectUriPath = this.getFullUrlPath(redirectUriParser);
                // watch for the redirect url
                let poolingInterval = setInterval(() => {
                    // if the popup closed
                    if (!popup || popup.closed || popup.closed === undefined) {
                        clearInterval(poolingInterval);
                        poolingInterval = null;
                        // enable login button
                        this.isLoginDisabled = false;
                    }
                    try {
                        // get the redirect url path
                        let popupWindowPath = this.getFullUrlPath(popup.location);
                        // if redirected successfully
                        if (popupWindowPath === redirectUriPath) {
                            if (popup.location.search || popup.location.hash) {
                                // parse the query string from the redirected url
                                let query = this.parseQueryString(popup.location.search.substring(1).replace(/\/$/, ''));
                                // handle the query data
                                this.handleRedirect(query)
                            } else {
                                console.log('OAuth redirect has occurred but no query or hash parameters were found.');
                            }
                            // clear the time interval
                            clearInterval(poolingInterval);
                            // close window
                            poolingInterval = null;
                            popup.close();
                            // enable login button
                            this.isLoginDisabled = false;
                        }
                    } catch (e) {
                        console.log('Error on authentication');
                    }
                }, 500);
            },
            /**
             * Generate full url path from given url
             * @param location
             * @returns {string}
             */
            getFullUrlPath: function (location) {
                let isHttps = location.protocol === 'https:';
                return location.protocol + '//' + location.hostname + ':' + (location.port || (isHttps ? '443' : '80')) + (/^\//.test(location.pathname) ? location.pathname : '/' + location.pathname);
            },
            /**
             * Parse the query string from url to object
             * @param str
             */
            parseQueryString: function (str) {
                let obj = {};
                let key;
                let value;
                (str || '').split('&').forEach((keyValue) => {
                    if (keyValue) {
                        value = keyValue.split('=');
                        key = decodeURIComponent(value[0]);
                        obj[key] = (!!value[1]) ? decodeURIComponent(value[1]) : true;
                    }
                });
                return obj;
            },
            /**
             * Handle the query data
             * @param query
             */
            handleRedirect: function (query) {
                if (query.hasOwnProperty('error')) {
                    console.log(query.error_reason);
                } else {
                    // TODO: Send ajax request to the server with 'query' variable
                    // This will contains the code required to preform the server side authentication
                }
            }
        }
    }
</script>

<style scoped>

</style>
