# notes

- browse_channels.tsx renders the channel browser
- this component doesn't use redux
- changeFilter handles filter change and updates the search term and sets filter state
- search function handles search
- setSearchResults function is where filtering seems to happen, at least for the channel types
- searchAllChannels function finds all channels and is in channels.ts

## Questions
- are you happy with filtering and sorting on the client side?
- are there situations where you shouldn't see channels that someone you dm has restricted access to?


## Steps

### finding users in social circle that are in sidebar and in dms
- sidebar_list.tsx has a prop that contains all the sidebar categories
- each category has a list of channel ids, so I can get the dms from here
- the function getCategoriesForCurrentTeam returns the categories
- will need to use or create a selector to get the user ids for the dms

### finding channels users are part of
- might need to make an api call to get the channels each user is in
- there is an api /api/v4/users/{user_id}/teams/{team_id}/channels" - name GetChannelsForTeamForUser
- this api gets all channels for a user within the team

- when creating the list of channels to display if paginated, in the func channelSearchQuery in channel_store.go
- first retrieve all channels that belong to social circle and paginate, if that runs out then retrieve the
user's unjoined channelSearchQuery

Possible ways to do this sort
1. query for all channels in user's social circle and in team, add a column - user_circle to sort by for overall query, then sort by most recent message, then most members
- query all channels sort by unjoined, will have to create new field in query to add unjoined property
- temp tables might be able to help with this to make querying faster, only useful while the connection is being used tho


### SELECT CHANNELS FROM SOCIAL circle
```sql
    SELECT channelid FROM channelmembers WHERE userid IN ('44aekhi34ffk3gto37ii1uek4c', 'qtzwymk897fx3kqw6p77a3uofe');
    
    -- gets channels that users are a member of and sorts them by last updated time
    SELECT channelid FROM channelmembers INNER JOIN channels ON channels.id = channelid WHERE userid IN ('44aekhi34ffk3gto37ii1uek4c', 'qtzwymk897fx3kqw6p77a3uofe') ORDER BY channels.lastpostat DESC;
    
    -- gets channels that users are a member of and sorts them by last updated time and returns a bool for if they are in 
    -- the users social circle, which they are by nature of this query
    SELECT channelid, channels.name, true as in_social_circle FROM channelmembers INNER JOIN channels ON channels.id = chann
    elid WHERE userid IN ('44aekhi34ffk3gto37ii1uek4c', 'qtzwymk897fx3kqw6p77a3uofe') ORDER BY channels.lastpostat DESC;
    
    -- joins channels with above query and sorts by in_social_circle and lastpostat
    WITH social_circle_channels AS (
        SELECT channelid, channels.name, true as in_social_circle
        FROM channelmembers INNER JOIN channels ON channels.id = channelid
        WHERE userid IN ('44aekhi34ffk3gto37ii1uek4c', 'qtzwymk897fx3kqw6p77a3uofe') ORDER BY in_social_circle, channels.lastpostat DESC
    ),
    unjoined_channels AS (
    SELECT distinct channelmembers.channelid, true as unjoined                        
        FROM channelmembers 
        WHERE channelmembers.channelid NOT IN (
            SELECT channelmembers.channelid
                FROM channelmembers
                WHERE '5z45bzixitrztbf8f9xyi1yz7h' = channelmembers.userid
            )
    ),
    public_joined_channels AS (
        SELECT channels.id, true as joined
        FROM channels
        LEFT JOIN channelmembers ON channelmembers.channelid = channels.id
        WHERE '5z45bzixitrztbf8f9xyi1yz7' = channelmembers.userid AND channels.type = 'O'
    )
    SELECT channels.name, social_circle_channels.in_social_circle, unjoined_channels.unjoined, public_joined_channels.joined
    FROM channels
    LEFT JOIN social_circle_channels ON channels.id = social_circle_channels.channelid
    LEFT JOIN unjoined_channels ON channels.id = unjoined_channels.channelid
    LEFT JOIN public_joined_channels ON channels.id = public_joined_channels.id
    ORDER BY social_circle_channels.in_social_circle, unjoined_channels.unjoined, public_joined_channels.joined, channels.lastpostat;
```



## TODO
- update all instances of formatMessage function or FormatMessage component ids to get translated strings
