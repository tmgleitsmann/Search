# Searching for Keywords in MongoDB Atlas Search
Up to this point in the repository, we've been talking about breaking up sequences of texts into arrays of "tokens". This is perfect if we want to query against a series of tokens with a set of tokens as input. But what if we want to match for exact terms?

This could be:
- usernames
- emails
- zip codes
- full names

Anything where instead of multiple tokens, we'll want to query for an individual token, without alteration.

> quick brown fox jumped over the lazy dog` would tokenize to `[The quick brown fox jumped over the lazy dog]` rather than traditionally `[quick, brown, fox, jump, over, laz, dog]`

