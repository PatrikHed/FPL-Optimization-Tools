# FPL Optimization Repository

This repository is a collection of optimization tutorials and recipes for Fantasy Premier League (FPL).

Python code mainly use `pandas` for data management and `sasoptpy` for optimization modeling.

It is being actively developed. The content and the structure of the repository might change.

## Tutorials

If you are interested in using optimization for FPL, see my YouTube tutorials on the subject.

### Python

Link: https://youtube.com/playlist?list=PLrIyJJU8_viOags1yudB_wyafRuTNs1Ed

Python tutorials include following topics

- Goalkeeper selection problem
- Single-period expected value maximization (squad, lineup, captain)
- Multi-period expected value maximization (squad, lineup, captain)
- Alternative solution generation
- Multi-objective optimization (2-Step and Weight methods)
- Bench decisions
- Auto-bench weights and iterative solution for nonlinear case
- Noise in expected values
- Sensitivity analysis
- Data collection from FPL API with login
- Wildcard (chip) optimization

## Excel

Link: https://youtube.com/playlist?list=PLrIyJJU8_viOLw3BovPDx5QLKkCb8XOTp

My Excel tutorials are rather short but might give you an idea what optimization is capable of doing.
Reach out to me if you need the raw data to give it a try.

- Goalkeeper selection problem
- Single-period expected value maximization (squad, lineup, captain)
- Multi-period expected value maximization (squad, lineup, captain)

## Instructions

### Steps

You will need to follow steps below to install required platform and also optimization solver (CBC).


- Download and install Python and Git to your machine
- Download CBC optimization solver binary and add it to your environment path (example: https://youtu.be/DFXCXoR6Dvw?t=1642)
- Clone the repository
  
  `git clone https://github.com/sertalpbilal/FPL-Optimization-Tools.git fpl-optimization`

- Install required packages
  
  ``` shell
  cd fpl-optimization
  python -m pip install -r requirements.txt
  ```

- Navigate to `data` directory and copy login file without `sample` extension
  
  ``` shell
  cd data
  cp login.json.sample login.json
  ```

- Download FPLReview projections and save it under `data` and rename it to `fplreview.csv`

- Edit values inside `login.json` file:
  
  ``` json
  {
    "email": "myemail@email.com",
    "password": "mypassword"
  }
  ```

  Note that logging in for Allsvenskan Fantasy does not yet work correctly, so you will have to manually get your team info and save it in a `team.json` file

- Navigate to `run` directory
  
  `cd ..\run`

  And run either `solve_regular.py` (for regular GW solve) or `solve_wildcard.py` (for wildcard optimization)  
  See instructions below.

### Multi-period (regular) GW optimization


- Edit content of `regular_settings.json` file
  
  ``` json
    {
        "api_base": "https://fantasy.allsvenskan.se",
        "xPts_file_path": "../data/allsvenskan_xpts.csv",
        "team_json": "../data/allsvenskanteam.json",
        "login_json": "../data/allsvenskanlogin.json",
        "login_url": "https://fantasy.allsvenskan.se/api/player/login/",
        "horizon": 5,
        "ft_value": 1.5,
        "itb_value": 0.2,
        "no_future_transfer": false,
        "randomized": false,
        "banned": [],
        "locked": [],
        "delete_tmp": true,
        "secs": 300,
        "use_cmd": false,
        "use_wc": null,
        "use_bb": null,
        "use_fh": null,
        "use_lr": null,
        "use_2c": null,
        "use_ptb": null,
        "chip_limits": {"bb": 0, "wc": 0, "fh": 0, "tc": 0, "lr":0, "2c":0, "ptb":0},
        "booked_transfers": []
    }
  ```
  - `api_base`: base url of the API. defaults to `https://fantasy.premierleague.com`
  - `xPts_file_path`: relative path to your xPts file. defaults to `../data/fplreview.csv`
  - `team_json`: relative path to your team.json file. defaults to `../data/team.json`
  - `login_json`: relative path to your team.json file. defaults to `../data/login.json`
  - `login_url`: url to the login endpoint. defaults to `https://users.premierleague.com/accounts/login/`
  - `horizon`: length of planning horizon
  - `ft_value`: value assigned to the extra free transfer
  - `itb_value`: value assigned to having 1.0 extra budget
  - `no_future_transfer`: `true` or `false` whether you want to plan future transfers or not
  - `randomized`: `true` or `false` whether you would like to add random noise to EV
  - `banned`: list of banned player IDs
  - `locked`: list of player IDs to always have during the horizon (e.g. `233` for Salah)
  - `delete_tmp`: `true` or `false` whether to delete generated temporary files after solve
  - `secs`: time limit for the solve (in seconds)
  - `use_cmd`: whether to use `os.system` or `subprocess` for running solver, default is `false`
  - `use_wc`: specify a gameweek where the solver has to use Wildcard (requires chip_limit for wc > 0)
  - `use_bb`: specify a gameweek where the solver has to use Bench Boost (requires chip_limit for bb > 0)
  - `use_fh`: specify a gameweek where the solver has to use Free Hit (requires chip_limit for fh > 0)
  - `use_tc`: specify a gameweek where the solver has to use Triple Captain (requires chip_limit for tc > 0)
  - `use_lr`: specify a gameweek where the solver has to use Loan Rangers (requires chip_limit for lr > 0)
  - `use_2c`: specify a gameweek where the solver has to use Two Captains (requires chip_limit for 2c > 0)
  - `use_ptb`: specify a gameweek where the solver has to use Park The Bus (requires chip_limit for ptb > 0)
  - `chip_limits`: sets the maximum limit for chip usage. (if you're optimizing from GW1 you will need to add +1 to wc)
  - `booked_transfers`: list of booked transfers for future gameweeks. needs to have a `gw` key and at least one of `transfer_in` or `transfer_out` with the player ID  (e.g. `233` for Salah)

- Run the multi-period optimization
  
  ``` shell
  python solve_regular.py
  ```

- Find the optimal plans under `run\results` directory with timestamp
  
  ```
    > cd results
    > ls
    regular_2021-11-04_10-00-00.csv
  ```



### Wildcard optimization


- Edit content of `wildcard_settings.json` file
  
  ``` json
    { 
        "horizon": 4,
        "use_wc": 8,
        "no_future_transfer": true,
        "randomized": false,
        "wc_limit": 1,
        "banned": [],
        "locked": [],
        "delete_tmp": true,
        "secs": 120
    }
  ```

  - `use_wc`: GW number you want to use your wildcard (use `null` if you want optimization to choose it for you)
  - `wc_limit`: 1 or 0, depending on you want to use WC chip or not

- Run the wildcard optimization
  
  ``` shell
  python solve_wildcard.py
  ```

- Find the optimal plans under `run\results` directory with timestamp
  
  ```
    > cd results
    > ls
    wildcard_2021-10-04_10-49-07.csv  wildcard_2021-10-04_10-54-50.csv
  ```

# License

[Apache-2.0 License](LICENSE)
