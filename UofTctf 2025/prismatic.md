There are 2 endpoints a user can go to.
First being the login page - 
```js
app.post(
    "/api/login",
    async (req, res) => {
        try {
            let {name, password} = req.body;
            let user = await prisma.user.findUnique({where:{
                    name: name
                },
                include:{
                    posts: true
                }
            });
            if (user.password === password) { 
                res.json({success: true, posts: user.posts});
            }
            else {
                res.json({success: false});
            }
        } catch (error) {
            res.json({success: false, error});
        }
    }
)
```
Here the user can simply enter a username and password. If the username and corresponding password is correct, then we are given access to the posts of the given user.

The second endpoint is the visible blog posts made by all users - 
```js
app.get(
  "/api/posts",
  async (req, res) => {
    try {
      let query = req.query;
      query.published = true;
      let posts = await prisma.post.findMany({where: query});
      res.json({success: true, posts})
    } catch (error) {
      res.json({ success: false, error });
    }
  }
);
```

All blog posts are created and have its visibility status set in `seed.js` file.
In that file, we are only concerned for 1 post, which is - 
```js
{
    title: `The Flag`,
    body: `This is a secret blog I am still working on. The secret keyword for this blog is ${FLAG}`,
    authorId: Math.floor(Math.random()*NUM_USERS)+1,
    published: false
}
```

There are 2 challenges posed here - we don't know which user has the password and neither do we know the password of any of the users.

```js

const USERS = [
    {
        name: "White",
        password: generateString(Math.floor(Math.random()*10)+15),
    },
    {
        name: "Bob",
        password: generateString(Math.floor(Math.random()*10)+15),
    },
    {
        name: "Tommy",
        password: generateString(Math.floor(Math.random()*10)+15),
    },
    {
        name: "Sam",
        password: generateString(Math.floor(Math.random()*10)+15),
    },
];
```
From this code, we can understand that the password can be of any length ranging from 15 to 24. This clue also hints to the fact that maybe we are meant to guess the password character by character.

In the post api, we can see this line - 
```js
  let posts = await prisma.post.findMany({where: query});
```
there is no filtering done to the `query` variable. Hence, we can put any query we want.
I tried directly doing `/api/posts?published=False` but this doesn't work for 2 reasons - 
1. 'False' over here would be considered as a string and not as a boolean value.
2. The `published` value gets overriden in the code and is always set to only allowing us to view when it is true.

But, we are able to get details such as the blogs only published by Bob by doing `/api/posts?author[name]=Bob`
From here, we just need to guess the password for each user by doing - 
`/api/posts?author[name]=Name&author[password][gte]=GuessedEndingPartOfPassword`
By writing a script, we can quickly get the password for all the users.
Finally by logging in to the correct user, we can get the flag.

