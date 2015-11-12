© (C) Copyright 2015 Travis Rigg

© Copyright phpBB Limited 2003-2014

This project is distributed under the terms of the GNU General Public License v2

This file is part of SFIM.

    SFIM is free documentation: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 2 of the License, or
    (at your option) any later version.

    SFIM is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with SFIM.  If not, see <http://www.gnu.org/licenses/>.

# Textual Entity Relational Model
After reassessing the project, I have determined that it can no longer be a goal
of SFIM to be a drop in replacement for another forum software. Instead I will
try to describe a small core that is completely necessary and can be extended
later with important features.

## Tables
### 1. forums
The forums table is used to describe different categories of discussion.

#### Keys:
	forum_id	unsigned int	Primary Key
	parent_id	unsigned int	Foreign	References the forums table.
	title			varchar				Unique	What is this forum called?

The structure of this table allows forums to be broken down further to create
categories for threads.

### 2. topics
The topics table is used to describe different conversations.

#### Keys:
	topic_id	unsigned int	Primary	Key														auto_increment
	forum_id	unsigned int	Foreign	References the forums table.

#### Attributes:
	start_date		timestamp	When was this topic started?
	subject				varchar		What is this topic about?
	sticky_status	boolean		Is this topic sticky?				false

### 3. posts
The topics table is used to describe what people have said in various topics.

#### Keys:
	post_id		unsigned int	Primary	Key
	topic_id	unsigned int	Foreign	References the topics table.
	user_id		unsigned int	Foreign	References the people table.

#### Attributes:
	posted_date	timestamp	When was the post made?
	message			text			What is being said?

### 4. people
The people table is used to describe users that are making posts and topics to
the forum.

#### Keys:
	user_id		unsigned int	Primary	Key
	username	varchar				Unique	The public name the user is known by.

#### Attributes:
	password	char	A password hash stored via the SHA512-Crypt algorithm.
	avatar		char	The name of the users avatar in the filesystem.
	signature	text	What does the user want to say at the bottom of all posts.

### 5. bans
The bans table represents the people who have been banned. This is a seperate
entity because bans are timed, and peoples user rights can be restored. However,
it might be useful to be able to count how many times a person has been banned
from interacting fully with the community. For example, an administrator might
want to set a rule that if a person gets banned three times their account is
permanently banned.

#### Keys:
	ban_id	unsigned int	Primary	Key
	user_id	unsigned int	Foreign	References the people table

#### Attributes:
	start_date	date		When did the user get banned?
	end_date		date		When will the user be re-enabled.
	reason			varchar	Why is the user being banned.

### 6. groups
The groups table represents user groups who can be given special priveleges. For
example, moderators.

#### Keys:
	group_id	unsigned int	Primary	Key
	title			varchar				Unique	What is this group called?

### 7. memberships
This is a relational entity that describes which people are members of which
groups. As such it is a weak entity and does not have a true primary key.
Instead the combination of a pair of keys are used as a primary key.

#### Keys:
	user_id		unsigned int	Foreign	References the people table.
	group_id	unsigned int	Foreign	References the groups table.

### 8. powers
This table describes what powers can be available to users.

#### Keys:
	power_id	unsigned int	Primary	Key
	title			varchar				Unique	What is the name of this power?

#### Attributes:
	description	text	What does the power actually entail?

### 9. permissions
This table is a relation entity that describes which groups have which powers.
For this reason it is a weak entity and does not have a true primary key.
Instead the comination of a pair of keys are used as a primary key.

#### Keys:
	group_id	unsigned int	Foreign	References the groups table.
	power_id	unsigned int	Foreign	References the powers table.

## Future features:
The above should be considered a minimum base description of SFIM. As the
standard lives on, more features will be added as extended levels of
functionality. The above will be known as "level 0", describing the minimum.
Implementers of the SFIM standard are discouraged from picking and choosing
features in higher levels. Instead, SFIM implementers should inform users of
their implementation of what level of SFIM they have implemented. This way a
user with a level 3 android app can know for sure that they can interact with
level 3 SFIM implemenations without having to worry about missing features or
anything breaking.

Now. Without further ado, future features:
* Private Messages
* Attachments
* Lightweight Markup Language
* Emoticons
* Polls
* Subscriptions
