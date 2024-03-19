# Proposal to improve Voter and Poll API using HATEOAS(Hypermedia as the Engine of Application State)

For better discoverability and decoupling of APIs we can use hypermedia concept to include links in response of Voter and Poll APIS

## New link struct

Link object will be used to add urls to an resouce. It is also a good idea to use a JSON object to represent a link. This gives us the option to add more information to links later.

```Go
//Link struct to represent hypermedia links
type Link struct {
    Rel  string `json:"rel"`  // Relationship type (self, poll, voter, etc.)
    Href string `json:"href"` // Hypermedia reference (URL)
    Type string `json:"type"` // MIME type of the linked resource
}
```

Now we add above link struct to existing structs like voter,poll, vote to include hypermedia links

- Modifcation to vote struct is as below

```Go
-   type Vote struct {
    VoteID    uint   `json:"voteId"`
    VoterID   uint   `json:"voterId"`
    PollID    uint   `json:"pollId"`
    VoteValue uint   `json:"voteValue"`
    Links     []Link `json:"links"` // Adding hypermedia links
}
```

New vote JSON with hypermedia links

```Go
vote := Vote{
    VoteID:    1,
    VoterID:   123,
    PollID:    456,
    VoteValue: 1,
    Links: []Link{
        {
            Rel:  "self",
            Href: "/voters/123/polls/456/votes/1",
            Type: "application/json",
        },
        {
            Rel:  "poll",
            Href: "/voters/123/polls/456",
            Type: "application/json",
        },
        {
            Rel:  "voter",
            Href: "/voters/123",
            Type: "application/json",
        },
    },
}
```

- Update voter struct to below

```Go
type voterPoll struct {
  PollID   uint `json:"poll_id"`
  VoteDate time.Time `json:"vote_date"`
  Links     []Link `json:"links"` // Adding hypermedia links
}

type Voter struct {
  VoterID     uint `json:"voter_id"`
  FirstName   string `json:"first_name"`
  LastName    string `json:"last_name"`
  VoteHistory []voterPoll `json:"voter_history"`
  Links     []Link `json:"links"` // Adding hypermedia links
}
```

Example Voter JSON with urls

```JSON
  {
    "voter_id" : 123,
    "first_name" : "John",
    "last_name" : "Doe",
    "links" : {
      "rel" : "self",
      "href" : "/voter/123",
      "type" : "application/json"
    }
    "voter_history" : [
      "poll_id" : 1,
      "vote_date" :  "<current_date_time>",
      "links" : {
      "rel" : "self",
      "href" : "/voter/123/polls/1",
      "type" : "application/json"
    }
    ]
  }
```

## Benifits of using HATEOAS while desinging APIs

By designing an API using HATEOAS concepts we bring below advantages

- Better discoverability
- As response has information above how to get more information other object, there is less hardcoded logic in clients
- It is also a way to self document the APIs, and future proff them because we can modify and change links with affecting the clients
- By implementing links in Voter and Poll APIs, if a user polls for a vote information, by just looking at the response he will know how to get voter information and also polls informations.
- Same goes for Voter API, by getting voter information, user can knows how to get polls information as well. As said it improves discoverability and seld documents Voter and Poll API

## References
1. https://www.mscharhag.com/api-design/hypermedia-rest
2. https://www.iana.org/assignments/link-relations/link-relations.xhtml
3. https://stateless.co/hal_specification.html
