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
### 1. Forums
The forums table is used to describe different categories of discussion.

#### Keys:
	forum_id	unsigned int	Primary Key
	title			varchar				Unique	What is this forum called?

### 2. Topics
The topics table is used to describe different conversations.

#### Keys:
	topic_id	unsigned int	Primary	Key
	forum_id	unsigned int	Foreign	References the forums table.
	user_id		unsigned int	Foreign	References the people table.

#### Attributes:
	subject	varchar		What is this topic about?
	created	timestamp	When was the topic started?

### 3. Posts
The topics table is used to describe what people have said in various topics.

#### Keys:
	post_id		unsigned int	Primary	Key
	topic_id	unsigned int	Foreign	References the topics table.
	user_id		unsigned int	Foreign	References the people table.

#### Attributes:
	posted	timestamp	When was the post made?
	message	text			What is being said?

### 4. People
The people table is used to describe users that are making posts and topics to
the forum.

#### Keys:
	user_id		unsigned int	Primary	Key
	username	varchar				Unique	The public name the user is known by.

#### Attributes:
	password	varchar	SHA2 Hashed version of the user's password.
