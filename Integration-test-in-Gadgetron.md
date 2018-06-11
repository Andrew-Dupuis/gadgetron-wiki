### Introduction

Gadgetron comes with integration test cases and ground-truth datasets. The purpose is to make sure continuous development will not break the functionality of whole system. 

Typical use case is :

Modify the code --> load integration test --> run the tests --> if all success --> submit pull request
     |<------------------------------------------------------|

### Download test data

Suppose the Gadgetron code was cloned to ~/mrprogs/gadgetron

```
cd ~/mrprogs/gadgetron
cd test/integration
python3 get_data.py
```

This command will download integration test data from Azure cloud to ~/mrprogs/gadgetron/test/integration/data.

The test cases are listed in  ~/mrprogs/gadgetron/test/integration/cases.

### Run the tests

Suppose gadgetron and ismrmrd are installed at ~/local, there are different modes to run the tests:

1) Run all tests
```
# batch mode to run all tests
# this will start a gadgetron at port 9003 of localhost
cd ~/mrprogs/gadgetron/test/integration
python3 run_all_tests.py -I ~/local -G ~/local ./test_cases.txt
```

2) Run one test
```
cd ~/mrprogs/gadgetron/test/integration
python3 run_gadgetron_test.py -I ~/local -G ~/local ./cases/simple_gre.cfg
```

3) Run tests on locally started gadgetron
```
# suppose gadgetron has been started at localhost, port 9008
cd ~/mrprogs/gadgetron/test/integration
python3 run_gadgetron_test.py -I ~/local -G ~/local -p 9008 ./cases/simple_gre.cfg
```

4) Get the help of test scripts
```
cd ~/mrprogs/gadgetron/test/integration
python3 run_all_tests.py --help
python3 run_gadgetron_test.py --help
```