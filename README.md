# Email-Verification-Golang


Let's break down the provided Go code step by step to understand its functionality and purpose.

## Overview

This Go program checks the DNS records of a given domain for the presence of MX (Mail Exchange), SPF (Sender Policy Framework), and DMARC (Domain-based Message Authentication, Reporting & Conformance) records. It reads domain names from standard input and outputs the results in a CSV format.

## Code Breakdown

### Package and Imports

```go
package main

import (
	"bufio"
	"fmt"
	"log"
	"net"
	"os"
	"strings"
)
```

- **package main**: Defines the package as `main`, which is necessary for creating an executable program.
- **Imports**: The program imports several packages:
  - `bufio`: For buffered I/O operations.
  - `fmt`: For formatted I/O, such as printing.
  - `log`: For logging errors.
  - `net`: For network-related functions, particularly DNS lookups.
  - `os`: For operating system functionalities, like reading from standard input.
  - `strings`: For string manipulation functions.

### Main Function

```go
func main() {
	scanner := bufio.NewScanner(os.Stdin)
	fmt.Printf("domain,hasMX,hasSPF,sprRecord,hasDMARC,dmarcRecord\n")

	for scanner.Scan() {
		checkDomain(scanner.Text())
	}

	if err := scanner.Err(); err != nil {
		log.Fatal("Error: could not read from input: %v\n", err)
	}
}
```

- **Scanner Initialization**: A scanner is created to read from standard input (`os.Stdin`).
- **Header Output**: The program prints a header row for the CSV output indicating what each column represents.
- **Loop Through Input**: It enters a loop that reads each line (domain name) from the input until there are no more lines.
- **Error Handling**: After reading, it checks for any errors that may have occurred during input reading.

### Domain Checking Function

```go
func checkDomain(domain string) {
	var hasMX, hasSPF, hasDMARC bool
	var spfRecord, dmarcRecord string
```

- **Function Definition**: This function takes a single argument `domain` which is a string representing the domain to be checked.
- **Variable Declaration**: Several boolean flags and string variables are declared to store results about MX, SPF, and DMARC records.

#### MX Record Lookup

```go
	mxRecords, err := net.LookupMX(domain)
	if err != nil {
		log.Printf("Error: %v\n", err)
	}
	if len(mxRecords) > 0 {
		hasMX = true
	}
```

- **Lookup MX Records**: The function attempts to retrieve MX records for the domain using `net.LookupMX`.
- **Error Handling**: If an error occurs during the lookup, it logs the error.
- **Check Presence**: If any MX records are found (`len(mxRecords) > 0`), it sets `hasMX` to true.

#### SPF Record Lookup

```go
	txtRecords, err := net.LookupTXT(domain)
	if err != nil {
		log.Printf("Error:%v\n", err)
	}

	for _, record := range txtRecords {
		if strings.HasPrefix(record, "v=spf1") {
			hasSPF = true
			spfRecord = record
			break
		}
	}
```

- **Lookup TXT Records**: The function retrieves TXT records for the domain using `net.LookupTXT`.
- **Error Handling**: Any errors encountered during this lookup are logged.
- **Check for SPF Record**: It iterates through the TXT records to find one that starts with "v=spf1", indicating it's an SPF record. If found, it sets `hasSPF` to true and stores the record.

#### DMARC Record Lookup

```go
	dmarcRecords, err := net.LookupTXT("_dmarc." + domain)
	if err != nil {
		log.Printf("ErrorL%v\n", err)
	}

	for _, record := range dmarcRecords {
		if strings.HasPrefix(record, "v=DMARC1") {
			hasDMARC = true
			dmarcRecord = record
			break
		}
	}
```

- **Lookup DMARC Records**: The function looks up TXT records specifically for DMARC by querying "_dmarc." + domain.
- **Error Handling**: Any errors during this lookup are logged as well.
- **Check for DMARC Record**: Similar to SPF, it checks if any of the retrieved records start with "v=DMARC1". If found, it sets `hasDMARC` to true and stores the record.

### Output Results

```go
	fmt.Printf("%v, %v, %v, %v, %v, %v", domain, hasMX, hasSPF, spfRecord, hasDMARC, dmarcRecord)
}
```

Finally, the function prints out all collected information in CSV format:
- Domain name
- Presence of MX records (`hasMX`)
- Presence of SPF records (`hasSPF`)
- The actual SPF record if present (`spfRecord`)
- Presence of DMARC records (`hasDMARC`)
- The actual DMARC record if present (`dmarcRecord`)

## Conclusion

This Go program is a simple yet powerful tool for checking essential DNS records related to email authentication. By using standard input and output in CSV format, it allows users to easily assess multiple domains in one go. Error handling ensures that issues during DNS lookups do not crash the program but are instead logged for review.
