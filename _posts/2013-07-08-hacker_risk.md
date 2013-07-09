---
title: Hacker Risk™
layout: post
---

I finally felt like I had reached a point working on Enclav.es where I had learned what I wanted to learn from the project (what a large-ish full-stack project would feel like from scratch to a running project) and while I hope to continue to work on it after Hacker School, I've decided that my time would be better spend on other things. Thus, I'm in the process of learning Haskell, and I've been helping some fellow Hacker Schoolers work on an implementation of Risk as a server so that anyone in the batch can write a standalone Risk™ AI to compete with anyone else in the batch who desires to write one. We're going to run a bracket, and whatnot. It should be fun.

Anyway, it's nearing completion, and while I don't feel that I've made a large number of contributions, today I implemented a full feature over the course of an hour or two (before we continued discussion of how to proceed, and also before pairing on the documentation of the API for a little while). 

It's nothing much, but the [main commit is here](https://github.com/eriktaubeneck/hacker_risk/commit/732b9bd959b1519cef4a43d13cb53e603e62e9ba).

I guess I just really like how simple this turned out. We had a few objects and wanted to be able to send them as JSON to the clients connecting to our server, so that we could serialize the entirety of the game's state into a single human-and-machine-friendly object, and Python made this super, super easy.

In the final version, the feature exists as two blocks of code. This is the first block, which is the method that actually gets called to generate the JSON. This is really simple, we build the very outer layer of the object we want to send (two objects: one representing the player's state and the other representing the public game state), and that's all that's needed here.

{% highlight python %}
    """
    Returns game state as JSON for sending to clients.
    Where g = Game(*args), 
    g.game_state_json(player)
    returns a JSON object of the entire game state as visible to player,
    where player is the requesting player
    """
    game_state = {'game':{},'you':{}}
    game_state['game']['continents'] = {}
    for key in self.board.continents:
    		continent = self.board.continents[key]
    		game_state['game']['continents'][key] = continent
    
    game_state['you'] = player
    return json.dumps(game_state, cls=GameEncoder)
{% endhighlight %}

Then I define the custom JSON encoder that's referred to in the last line of the above snippet, where `cls=GameEncoder`. Python allows me to extend the parent class to define JSON serialization for whenever `json.dumps` finds one of our custom classes. Again, it's simple, but that's why I like it, I think.

{% highlight python %}
class GameEncoder(json.JSONEncoder):
    """Special JSON encoder for our objects"""
    def default(self, obj):
        if isinstance(obj, models.Country):
            return { 'owner':obj.owner,
                    'troops':obj.troops
            }

        elif isinstance(obj, models.Player):
            return { 'is_eliminated':obj.is_eliminated,
                     'cards':list(obj.cards),
                     'earned_cards_this_turn':obj.earned_card_this_turn,
                     'countries':[ country.name for country in obj.countries ],
                     'troops_to_deploy':obj.troops_to_deploy
            }

        elif isinstance(obj, models.Continent):
            return {'countries': obj.countries, 'bonus':obj.bonus}

        elif isinstance(obj, models.Card):
            return { 'country':obj.country, 'value':obj.value }

        else: 
            return json.JSONEncoder.default(self, obj)
{% endhighlight %}

By simply defining the JSON serialization for these custom objects (and notice that in some cases they reference each other, but that this implementation entirely allows that), I'm able to construct what is ultimately a rather large but organized data structure that contains all of the game's state without any really confusing code at all. Pretty cool!

The empty version of the data structure from my tests (varies slightly from the version above since this is from the version in the original commit) looks like this:

{% highlight javascript %}
    {
        "game": {
            "continents": {
                "europe": {
                    "bonus": 5,
                    "countries": {
                        "northern europe": {
                            "owner": null,
                            "troops": 0
                        },
                        "iceland": {
                            "owner": null,
                            "troops": 0
                        },
                        "southern europe": {
                            "owner": null,
                            "troops": 0
                        },
                        "great britain": {
                            "owner": null,
                            "troops": 0
                        },
                        "western europe": {
                            "owner": null,
                            "troops": 0
                        },
                        "scandanavia": {
                            "owner": null,
                            "troops": 0
                        },
                        "russia": {
                            "owner": null,
                            "troops": 0
                        }
                    }
                },
                "australia": {
                    "bonus": 2,
                    "countries": {
                        "new guinea": {
                            "owner": null,
                            "troops": 0
                        },
                        "eastern australia": {
                            "owner": null,
                            "troops": 0
                        },
                        "western australia": {
                            "owner": null,
                            "troops": 0
                        },
                        "indonesia": {
                            "owner": null,
                            "troops": 0
                        }
                    }
                },
                "africa": {
                    "bonus": 3,
                    "countries": {
                        "madagascar": {
                            "owner": null,
                            "troops": 0
                        },
                        "east africa": {
                            "owner": null,
                            "troops": 0
                        },
                        "south africa": {
                            "owner": null,
                            "troops": 0
                        },
                        "egypt": {
                            "owner": null,
                            "troops": 0
                        },
                        "north africa": {
                            "owner": null,
                            "troops": 0
                        },
                        "central africa": {
                            "owner": null,
                            "troops": 0
                        }
                    }
                },
                "asia": {
                    "bonus": 7,
                    "countries": {
                        "afghanistan": {
                            "owner": null,
                            "troops": 0
                        },
                        "india": {
                            "owner": null,
                            "troops": 0
                        },
                        "irkutsk": {
                            "owner": null,
                            "troops": 0
                        },
                        "yakutsk": {
                            "owner": null,
                            "troops": 0
                        },
                        "southeast asia": {
                            "owner": null,
                            "troops": 0
                        },
                        "mongolia": {
                            "owner": null,
                            "troops": 0
                        },
                        "china": {
                            "owner": null,
                            "troops": 0
                        },
                        "kamchatka": {
                            "owner": null,
                            "troops": 0
                        },
                        "japan": {
                            "owner": null,
                            "troops": 0
                        },
                        "ural": {
                            "owner": null,
                            "troops": 0
                        },
                        "siberia": {
                            "owner": null,
                            "troops": 0
                        },
                        "middle east": {
                            "owner": null,
                            "troops": 0
                        }
                    }
                },
                "north america": {
                    "bonus": 5,
                    "countries": {
                        "ontario": {
                            "owner": null,
                            "troops": 0
                        },
                        "eastern canada": {
                            "owner": null,
                            "troops": 0
                        },
                        "eastern united states": {
                            "owner": null,
                            "troops": 0
                        },
                        "alaska": {
                            "owner": null,
                            "troops": 0
                        },
                        "central america": {
                            "owner": null,
                            "troops": 0
                        },
                        "greenland": {
                            "owner": null,
                            "troops": 0
                        },
                        "western united states": {
                            "owner": null,
                            "troops": 0
                        },
                        "alberta": {
                            "owner": null,
                            "troops": 0
                        },
                        "northwest territory": {
                            "owner": null,
                            "troops": 0
                        }
                    }
                },
                "south america": {
                    "bonus": 2,
                    "countries": {
                        "brazil": {
                            "owner": null,
                            "troops": 0
                        },
                        "argentina": {
                            "owner": null,
                            "troops": 0
                        },
                        "venezuela": {
                            "owner": null,
                            "troops": 0
                        },
                        "peru": {
                            "owner": null,
                            "troops": 0
                        }
                    }
                }
            }
        },
        "you": {
            "cards": [],
            "earned_cards_this_turn": false,
            "is_eliminated": false,
            "countries": []
        }
    }
{% endhighlight %}

