---
layout: post
title:      "The Anti-Social Network"
date:       2018-02-26 18:27:59 +0000
permalink:  the_anti-social_network
---


![](https://imgur.com/mu94k6q.png)

Welcome, to the Anti-Social Network! As this is my final project in this course, It is time to pull together all of the various skills that have been developing over the last several projects, and package them together in a unique presentation. The core of this application will be a Rails API backend and a React JS frontend with Redux middleware. The last blog post goes in depth about setting up the two servers in such a way they can communicate, this post is going to focus on making the actual application work.

The inspiration for the Anti-Social Network comes from what I believe to be a fundamental flaw in social medias, namely, the ability to contribute an unending amount of content. Then the endless political and social carnage ensues as we fight and argue via proxies, constantly shouting into the electronic ether. Not great. At the Anti-Social Network, your ability to post content is limited daily, and hopefully as a result, a greater emphasis is placed on the quality of those contributions. 


# Authentication
First things first thought, if we are designing our own social media network, we are going to need users, and for that we need authentication. Now, persisting your own passwords is always a risk, and this already feels like a niche site, so I'm going to use GitHub as my login credentials, so all our developer friends can come join the new social network!

GitHub uses an authentication system known as JWT or JSON Web Tokens. You can read more about their implementation [here.](https://developer.github.com/apps/building-oauth-apps/) Essentially, you log into GitHub and you are granted a Token that validates the user and user's actions. This token is then included in the headers for GET and POST requests to our backend, that way we never have to touch the user's actual credentials, but can authenticate their every action. Here is the front end code that fetches and sets our token: 

```
export default class AuthService {
  doAuthentication(token) {
    // Saves the user token
    this.setToken(token);
    // navigate to the home route
    window.location.replace('/home');
  }

  loggedIn() {
    // Checks if there is a saved token and it's still valid
    return !!this.getToken();
  }

  setToken(idToken) {
    // Saves user token to local storage
    sessionStorage.setItem('id_token', idToken);
  }

  getToken() {
    // Retrieves the user token from local storage
    return sessionStorage.getItem('id_token');
  }

  logout() {
    // Clear user token and profile data from local storage
    sessionStorage.removeItem('id_token');
    window.location.replace('/');
  }

  getQueryParams() {
    const query = window.location.search.substring(1);
    const pairs = query.split('&').map(str => str.split('='));
    return pairs.reduce((memo, pair) => {
      memo[pair[0]] = pair[1];
      return memo;
    }, {});
  }
}
```

This is the backend: 

```
class Authenticator
  def initialize(connection = Faraday.new)
    @connection = connection
  end

  def github(code)
    access_token_resp = fetch_github_access_token(code)
    access_token = access_token_resp['access_token']
    user_info_resp = fetch_github_user_info(access_token)
    {
      issuer: ENV['ANTISOCIALNETWORK_CLIENT_URL'],
      login: user_info_resp['login'],
      name: user_info_resp['name'],
      avatar_url: user_info_resp['avatar_url'],
      bio: user_info_resp['bio']
    }
  end

  private

  def fetch_github_access_token(code)
    resp = @connection.post ENV['GITHUB_ACCESS_TOKEN_URL'], {
      code:          code,
      client_id:     ENV['CLIENT_ID'],
      client_secret: ENV['CLIENT_SECRET']
    }
    raise IOError, 'FETCH_ACCESS_TOKEN' unless resp.success?
    URI.decode_www_form(resp.body).to_h
  end

  def fetch_github_user_info(access_token)
    resp = @connection.get ENV['GITHUB_USER_INFO_URL'], {
      access_token: access_token
    }
    raise IOError, 'FETCH_USER_INFO' unless resp.success?
    JSON.parse(resp.body)
  end
end
```

```
module TokiToki
  def self.encode(sub)
    payload = {
      iss: ENV['ANTISOCIALNETWORK_CLIENT_URL'],
      sub: sub,
      exp: 4.hours.from_now.to_i,
      iat: Time.now.to_i
    }
    JWT.encode payload, ENV['JWT_SECRET'], 'HS256'
  end

  def self.decode(token)
    options = {
      iss: ENV['ANTISOCIALNETWORK_CLIENT_URL'],
      verify_iss: true,
      verify_iat: true,
      leeway: 30,
      algorithm: 'HS256'
    }
    JWT.decode token, ENV['JWT_SECRET'], true, options
  end
end
```

This is a lot, but at its core it's just a simple exchange of credentials. We include the token when we need to access our backend by pulling it from sessionStorage, or preferably a cookie. The backend then decodes our token, and performs the desired response. All in all, this is far superior to handling our own database of users credentials, gross.
# Routes and Views
Now that we have users, we need somewhere for them to go and something for them to do. React as a framework approaches building websites a little differently. Instead of having a series of different web pages linked together, React actually structures the entire site in essentially one window, routing to the different views within that environment. Redux, our middleware, assists in that by providing us a Router. This is what our routes look like in our index.js file:

```
<Router>
        <App>
          <Switch>
            <Route path="/auth" component={LoginTransition} />
            <Route path="/logout" component={LogoutTransition} />
            <Route path="/home" component={UserProfile} />
            <Route path="/users/:userId" component={UserShow} />
            <Route path="/users" component={UsersList} />
            <Route path="/posts" component={Posts} />
            <Route path="/postFeed" component={UserConnectionsPosts} />
            <Route exact path="/" component={Login} />
            <Route path="*" component={NotFound} />
          </Switch>
        </App>
      </Router>
```

Things to note here, App is our actual application and is responsible for rendering all of its 'children' which in this case, are all of the Route's listed inside it. Naturally all of the Route's are also wrapped in our Router component, that just makes sense. That last component is the Switch component. It allows us to render only the specific Route that we want. Without it, any route which a matching path will be rendered all together, usually not what we want. There a couple of paths here that are unique. The top two are transition components, and give us a second to perform our authentication before taking the user where they want to go. The '/users/:userID' Route is a user's show page. The ':userID' param is then made available to the component so the correct user can be rendered. The last path with the star catches any other Route the user might enter, allowing us to redirect back to a valid component. 
# React Toolbox
Okay, so now we have an authentication system with users, we also have Routes set up that our users can navigate to different components. Now they need something to actually interact with. A few other basic social media things we need to accomplish is the ability to make connections. A user by themself posting content for only them isn't exactly the most creative or intriguing model.

Now, in order to provide this functionality, we need a user interface. For this project, because of the lightweightness and customizability of their framework, I chose [ReactToolbox.](http://react-toolbox.io/#/) Next thing next, we need to build a UserCard to display our users, pulling in some basic info from their github profile. This is our UserCard component:

```
class UserCard extends Component {
  componentDidMount() {
    this.props.getCurrentUser();
    this.props.getConnectionsIds();
  }

  addConnection = connection => {
    // Hides the clicked button
    connection.currentTarget.style.visibility = 'hidden';
    // Passes in the value of 'userId' of the clicked button
    this.props.createConnection(connection.currentTarget.attributes[0].value);
  };

  render() {
    return (
      <div className="user-card-container">
        <Card className="user-card" raised>
          <CardTitle
            className="user-card-title"
            avatar={logo}
            subtitle={this.props.user.name}
            title={this.props.user.login}
          />
          <CardMedia
            className="UserAvatar"
            aspectRatio="square"
            image={this.props.user.avatar_url}
          />
          <CardText>{this.props.user.bio}</CardText>
          <div className="user-card-actions">
            <CardActions>
              {this.props.connectionsIds.includes(this.props.user.id) ||
              this.props.user.id === this.props.currentUser.id ? (
                ''
              ) : (
                <Button
                  className="add-connection-button"
                  userid={this.props.user.id}
                  onClick={this.addConnection}
                >
                  Add Connection
                </Button>
              )}
            </CardActions>
          </div>
        </Card>
      </div>
    );
  }
}

const mapStateToProps = state => {
  return {
    currentUser: state.userReducer.currentUser,
    connectionsIds: state.connectionReducer.connectionsIds
  };
};

export default connect(mapStateToProps, {
  getConnectionsIds,
  createConnection,
  getCurrentUser
})(UserCard);
```

React Toolbox gives us the Card component and its subcomponents like CardTitle and CardActions. This allows us to build out a basic UI block while only having to consider the data. The UserCard is also connected to the Redux store because it has an Add Connection button, and it is responsible for appraising whether or not such a button should be rendered. One of our Routes rendered a collection of these UserCards so that new connections can be formed. Connections are made available in a SidePanel in the Application Layout (another component borrowed from ReactToolbox) and this is what that looks like, p.s. all the User seed data uses the [Faker Gem](https://github.com/stympy/faker): 

![](https://imgur.com/pJ7sbDq.png)

This is the async action triggered by the 'Add Connection' button: 

```
export const createConnection = connection => {
  return dispatch => {
    return fetch(`${API_URL}/connections`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `${token}`
      },
      body: JSON.stringify({ connection: connection })
    })
      .then(response => response.json())
      .then(connection => {
        dispatch(addConnection(connection));
      })
      .catch(error => console.log(error));
  };
};
```
# Post Submission
Fantastic! We now have authenticated users forming connections by navigating our routes, interacting with our components, and subsequently sending all of that information back and forth between our two servers with a JWT token. Now we just have to handle the actual creation of Posts and whatnot. 

First things first, we need a PostCard to hold the content. The PostCard itself can be stateless, as it is just vessel for data, and doesn't have to update itself, or consider the data it is rendering. This should do that:

```
const PostCard = ({ post }) => (
  <div className="post-card-container">
    <div className="post-content">
      <Card className="post-card">
        <CardTitle
          avatar={post.user.avatar_url}
          title={post.user.login}
          subtitle={post.user.name}
        />
        <CardMedia
          aspectRatio="wide"
          image={`${localHost}${post.picture.url}`}
        />
        <CardTitle title={post.title} />
        <CardText>{post.content}</CardText>
      </Card>
    </div>
    <div className="comments" />
  </div>
);

export default PostCard;
```

We are using the same bas Card component as for our users, which will give us a little consistency in design, and results in a post like this:

![](https://imgur.com/kf3BISh.png)

Now, to get to this point, we are going to need a form, a corresponding form reducer, actions and a list of posts to inject into. There is also the case of the image. Uploading simple string of text to our Rails server is no big deal, images requires a little work. There are a variety of gems that can assist in this, and I chose [CarrierWave.](https://github.com/carrierwaveuploader/carrierwave) What makes CarrierWave a good choice is it naturally is set up to work with base64 encoding. What is that? Well, we need to encode the image in some format so that it can be sent across the web and instantiated in our database. Base64 is a standard for doing just that. A base64 string looks a little like this, only much, much longer:

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAUIA....
```

CarrierWave will provide a PictureUploader that will then decode the base64 string and save the image in the database, such that it can be fetched for future renders. All in all, when implemented with the correct utilities, submitting images to the backend is about as painless as our basic strings. 
# Conclusion
And here is our User Profile, fully equipped with a UserCard, a PostForm with image upload and a list of UsersPosts!

![](https://imgur.com/sYpSs3a.png)

So after all that, where are we. We used GitHub's JWT authentication system, to validate users in our Rails API server so that we could render our ReactToolbox built components within our Redux routers views, submitting base64 encoded images to our backend of user connections in our brand new, cutting edge social media network. There are several advanced techniques that help bring all of this together, and even more hours playing around with CSS and Javascript effects. In the end, we have a fully functional app, built from the ground up in such a way that lends itself to further expansion. Essentially, we just dipped our toe in the great wide ocean of Web Development!


Project Links:

[API](https://github.com/dkStephanos/anti-social-network-api)

[Client](https://github.com/dkStephanos/anti-social-network-client)

